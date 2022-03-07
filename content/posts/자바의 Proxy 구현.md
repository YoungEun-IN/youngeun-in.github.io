---
title: 자바의 Proxy 구현
date: 2022-03-07T15:12:26+09:00
categories:
  - java
tags: 
  - dynamic-proxy
  - cglib-library
---

## Pure Java Proxy

```java
public interface TargetObject { 
  String someMethod(String name); 
}
```

```java
public class TargetObjectImpl implements TargetObject { 
  @Override 
  public String someMethod(String name) { 
    return "Real Subject method "+ name +"\n"; 
  } 
}
```

```java
public class TargetObjectProxy implements TargetObject { 
  TargetObjectImpl subject; 
  subject = new TargetObjectImpl(); 
  
  @Override 
  public String someMethod(String name) { 
    System.out.println("Before proxy"); 
    return subject.someMethod(name) + ("After proxy\n"); 
  } 
}
```

```java
public class PureProxyClient {
    public static void main(String[] args) {
        PureProxyClient client = new PureProxyClient();
        client.run("Lob");
    }

    public void run(String name) {
        TargetObjectProxy proxy = new TargetObjectProxy();
        System.out.println(proxy.someMethod(name));
    }
}
```

결과는 다음과 같다.
```
Before proxy 
Real Subject method Lob 
After proxy
```

## JDK Dynamic Proxy
Java 에서 지원하는 동적(런타임) Proxy 구현 방식이다.
특정 인터페이스들을 구현하는 클래스나 인스턴스를 만드는 기술이다.
Reflection API을 사용하여 Target Class의 method를 invoke()를 통해 동작시킨다.

Advise 대상이든 아니든 모든 Method Call 마다 reflection API의 invoke를 실시한다는 단점이 있다.
또한 인터페이스 기반의 Proxy → 모든 Target Class 는 Interface를 implement 하고 있어야 한다는 제약이 있다.

Spring AOP ProxyFactory에서 사용한다.

```java
public interface TargetObject { 
  void someMethod(String name); 
}
```

```java
public class TargetObjectImpl implements TargetObject {
    @Override
    public void someMethod(String name) {
        System.out.println("Real Subject Do something " + name);
    }
}
```

```java
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class DynamicProxyClient {
    public static void main(String[] args) {
        DynamicProxyClient dynamicProxyClient = new DynamicProxyClient();
        dynamicProxyClient.run("Lob");
    }

    public void run(String name) {
        realObject.someMethod(name);
    }

    TargetObject realObject = (TargetObject) Proxy.newProxyInstance(TargetObject.class.getClassLoader(), new Class[]{TargetObject.class}, (InvocationHandler) (proxy, method, args) -> {
        System.out.println("Before Proxy");
        TargetObject targetObject = new TargetObjectImpl();
        Object invoke = method.invoke(targetObject, args);
        System.out.println("After Proxy");
        return invoke;
    });
}
```

결과는 다음과 같다.
```
Before Proxy 
Real Subject Do something Lob 
After Proxy
```

## CGLIB Library를 통한 Dynamic Proxy 구현 방식

실제 바이트 코드를 조작하여 JDK Dynamic Proxy 보다 상대적으로 빠르다. 그러나 final이나 private으로 선언되어 상속된 객체에 Overriding을 제공하지 않는 경우에는 해당 행위에 대해서 Aspect를 적용할 수 없다.

Spring Boot 에서는 CGLIB를 기반으로 Spring AOP를 지원한다.

```java
public class TargetObject {
    void someMethod(String name) {
        System.out.println("Real Subject Do something " + name);
    }
}
```

```java
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodProxy;

public class CglibDynamicProxyClient {
    public static void main(String[] args) {
        CglibDynamicProxyClient client = new CglibDynamicProxyClient();
        client.run("Lob");
    }

    public void run(String name) {
        MethodInterceptor methodInterceptor = new MethodInterceptor() {
            final TargetObject targetObject = new TargetObject();

            @Override
            public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
                System.out.println("Before Proxy");
                Object invoke = method.invoke(targetObject, args);
                System.out.println("After Proxy");
                return invoke;
            }
        };
        TargetObject targetObject = (TargetObject) Enhancer.create(TargetObject.class, methodInterceptor);
        targetObject.someMethod(name);
    }
}
```

결과는 다음과 같다.
```java
Before Proxy 
Real Subject Do something Lob 
After Proxy
```

## 출처
https://lob-dev.tistory.com/entry/Java%EC%97%90%EC%84%9C-%EA%B5%AC%ED%98%84%ED%95%A0-%EC%88%98-%EC%9E%88%EB%8A%94-Proxy-%EB%93%A4-Pure-JDK-CGLIB
