# 자바의 Proxy 구현


Pure Java, JDK Dynamic Proxy, CGLib Dynamic Proxy를 이용하여 문자열을 대문자로 변환하는 프록시를 구현해보도록 하자. 인터페이스와 타겟 클래스는 다음과 같다. 

```java
public interface Hello { 
  String sayHello(String name); 
  String sayHi(String name); 
  String sayThankyou(String name); 
}
```

```java
public class HelloTarget implements Hello {
  @Override 
  public String sayHello(String name) { 
    return "Hello " + name; 
  } 
  
  @Override 
  public String sayHi(String name) { 
    return "Hi " + name; 
  }
  
  @Override 
  public String sayThankyou(String name) { 
    return "Thankyou " + name; 
  } 
}
```

예상 결과는 다음과 같다.

```
HELLO YYOUNGEUN
HI YYOUNGEUN
THANKYOU YYOUNGEUN
```

## Pure Java Proxy

```java
public class HelloProxy implements Hello { 
  HelloTarget helloTarget = new HelloTarget(); 
  
  @Override 
  public String sayHello(String name) { 
    return helloTarget.sayHello(name).toUpperCase(); 
  }
  
  @Override 
  public String sayHi(String name) { 
    return helloTarget.sayHi(name).toUpperCase(); 
  } 
  
  @Override 
  public String sayThankyou(String name) { 
    return helloTarget.sayThankyou(name).toUpperCase(); 
  }   
}
```

```java
public class Main {
    public static void main(String[] args) {
        String name = "yyoungeun";

        HelloProxy proxy = new HelloProxy();
        System.out.println(proxy.sayHello(name));
        System.out.println(proxy.sayHi(name));
        System.out.println(proxy.sayThankyou(name));
    }
}
````

순수 Java Proxy 구현은 **인터페이스에 대해서 모든 메서드를 직접 구현해야 하며, 동일한 액션이 있을경우 중복이 발생한다**는 문제가 있다.

## JDK Dynamic Proxy
다이나믹 프록시는 `InvocationHandler`를 통해 위의 두 문제를 해결한다. Reflection API를 사용하는 invoke 메서드를 구현 함으로서 proxy를 수행한다. 핵심은 타깃의 인터페이스를 기준으로 Proxy를 생성해준다는 점이다. Spring AOP는 JDK Dynamic Proxy를 기반으로 AOP 기술을 구현하였다.

프록시 인스턴스를 만드는 메소드는 다음과 같다.

```
Object Proxy.newProxyInstance(ClassLoader, Interfaces, InvocationHandler)
```

```java
import java.lang.reflect.Proxy;

public class Main {
    public static void main(String[] args) {
        String name = "yyoungeun";

        Hello proxy = (Hello) Proxy.newProxyInstance(
                Main.class.getClassLoader(),
                new Class[]{Hello.class},
                new UpperHandler(new HelloTarget())
        );
   
        System.out.println(proxy.sayHello(name));
        System.out.println(proxy.sayHi(name));
        System.out.println(proxy.sayThankyou(name));
    }
}
```
* 첫 번째 인자: 프록시를 만들 클래스 로더
* 두 번째 인자: 어떤 인터페이스에 대해 프록시를 만들 것인지 명시
* 세 번째 인자: InvocationHandler 인터페이스의 구현체
* 리턴 값: 동적으로 만든 프록시 객체

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

@AllArgsConstructor
class UpperHandler implements InvocationHandler {

    Object target;

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object invoke = method.invoke(target, args);
        if (invoke instanceof String && method.getName().startsWith("say"))
            return ((String) invoke).toUpperCase();
        else
            return invoke;
    }
}
```

InvocationHandler는 invoke()라는 메소드 하나만 가지고 있는 인터페이스이다. invoke() 메소드는 다이나믹하게 생성될 프록시의 메소드가 호출됐을 때 호출되는 메소드로, 여기서 어떤 메소드의 기능을 확장할지 결정할 수 있고, 확장된 기능을 구현할 수 있다.

**JDK Dynamic Proxy는 Advise 대상이든 아니든 모든 Method Call 마다 reflection API의 invoke를 실시한다**는 단점이 있다.
또한 **인터페이스 기반의 Proxy이기 때문에 모든 Target Class는 Interface를 implement 하고 있어야 한다**는 제약이 있다.

## CGLIB(Code Generator Library)을 통한 Dynamic Proxy

**실제 바이트 코드를 조작하여 JDK Dynamic Proxy 보다 상대적으로 빠르다.** 그러나 **final이나 private으로 선언된 경우에는 해당 행위에 대해서 Aspect를 적용할 수 없다.** CGLib은 클래스를 상속받아 Proxy를 생성해 주므로 Spring Boot에는 CGLib 방식을 기본 Proxy 생성 방법으로 사용하고 있다.

```java
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

public class Main {
    public static void main(String[] args) {
        String name = "yyoungeun";

        MethodInterceptor methodInterceptor = new MethodInterceptor() {
            final HelloTarget helloTarget = new HelloTarget();

            @Override
            public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
                Object invoke = method.invoke(helloTarget, args);
                if (invoke instanceof String && method.getName().startsWith("say"))
                    return ((String) invoke).toUpperCase();
                else
                    return invoke;
            }
        };

        Hello proxy = (HelloTarget) Enhancer.create(HelloTarget.class, methodInterceptor);
        System.out.println(proxy.sayHello(name));
        System.out.println(proxy.sayHi(name));
        System.out.println(proxy.sayThankyou(name));

    }
}
```

## 참고
https://lob-dev.tistory.com/entry/Java%EC%97%90%EC%84%9C-%EA%B5%AC%ED%98%84%ED%95%A0-%EC%88%98-%EC%9E%88%EB%8A%94-Proxy-%EB%93%A4-Pure-JDK-CGLIB

https://live-everyday.tistory.com/217

https://gmoon92.github.io/spring/aop/2019/04/20/jdk-dynamic-proxy-and-cglib.html

