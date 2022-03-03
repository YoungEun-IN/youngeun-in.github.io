# 프록시 패턴 구현


## 프록시 패턴
Proxy는 우리말로 대리자, 대변인 이라는 뜻이다. 대리자, 대변인은 다른 누군가를 대신해서 그 역할을 수행하는 존재를 말한다.
프록시는 타겟의 기능을 확장하거나 타깃에 대한 접근을 제어하기 위한 목적으로 사용된다.

### 상속을 사용한 구현

아래와 같은 타겟 클래스가 있다고 하자.

```java
public class GameService {
    public void startGame() {
        System.out.println("Hello");
    }
}
```
다음과 같이 프록시 클래스가 타겟 클래스를 상속받도록 만든다.
```java
public class GameServiceProxy extends GameService {
    @Override
    public void startGame() {
        long before = System.currentTimeMillis();
        //상위 메소드 실행
        super.startGame();
        System.out.println(System.currentTimeMillis() - before);
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        GameService gameService = new GameServiceProxy();
        gameService.startGame();
    }
}
```

결과는 다음과 같다.
```
Hello
0
```

### 인터페이스를 사용한 구현

인터페이스를 사용할 경우, 좀 더 구조가 유연하며 테스트하기에도 쉽다.

다음과 같이 인터페이스를 선언한다.
```java
public interface GameService {
    void startGame();
}
```
인터페이스를 구현한 클래스를 만들고, 메소드를 오버라이드한다.
```java
public class DefaultGameService implements GameService{
    @Override
    public void startGame() {
        System.out.println("Hello");
    }
}
```

다음과 같이 프록시 클래스를 선언한다.
```java
@RequiredArgsConstructor
public class GameServiceProxy implements GameService {
    //GameService를 외부에서 주입받는다.
    private final GameService gameService;

    @Override
    public void startGame() {
        long before = System.currentTimeMillis();
        gameService.startGame();
        System.out.println(System.currentTimeMillis() - before);
    }
}
```

클라이언트에서는 GameService 구현체를 생성하여 프록시 객체에 넘겨준다.
```java
public class Client {
    public static void main(String[] args) {
        GameService gameService = new GameServiceProxy(new DefaultGameService());
        gameService.startGame();
    }
}
```

## 다이나믹 프록시
프록시 패턴의 단점은 다음과 같다.
1. 인터페이스를 직접 구현해야 한다.
2. 프록시 클래스 내에 중복이 발생한다.

위와 같은 문제는 다이나믹 프록시를 사용하여 해결할 수 있다.
Dynamic Proxy는 런타임 시점에 인터페이스를 구현하는 클래스 또는 인스턴스를 만드는 기술을 말한다.
다이나믹 프록시를 사용하면, 일일이 구현해야한다는 문제는 reflection API가 해결해주고, 중복은 InvocationHandler가 해결해준다. 

InvocationHandler의 invoke() 메소드에서 타겟의 어떤 메소드에 적용할지 검사하고, 적용해야 하는 메소드라면 타겟의 원래 메소드를 호출하여 결과를 담아둔 뒤 추가 기능을 적용하여 리턴한다. 적용하지 말아야 하는 메소드라면 타겟의 원래 메소드의 리턴을 그대로 리턴한다.
 
 ```java
BookService bookService = (BookService) Proxy.newProxyInstance(BookService.class.getClassLoader(), new Class[]{BookService.class},
        new InvocationHandler() {
            BookService bookService = new RealSubjectBookService();
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                if (method.getName() == "rent") {
                    System.out.println("11111");

                    Object invoke = method.invoke(bookService, args);

                    System.out.println("22222");


                    return invoke;
                }
                return method.invoke(bookService, args);
            }
        });
 ```
Proxy.newProxyInstance 메소드를 이용해 프록시 객체를 생성한다.
* 첫번째 인자는 프록시 객체를 생성할 인터페이스 타입의 클래스로더
* 두번째 인자는 해당 프록시가 어떤 인터페이스를 구현체인가
* 3번째 인자는 이를 구현하는 InvocationHandler 이다.

InvocationHandler는 invoke 메소드를 구현한다.
* 첫번째 인자는 newProxyInstance를 사용해 생성된 프록시 객체의 참조
* 두번째 인자는 프록시를 통해 호출된 메소드의 참조
* 세번째 인자는 해당 메소드의 파라메터들의 참조

invoke 메소드 내에서 method.invoke 를 통해 메소드를 호출할 수 있다.
만약 프록시 객체를 통해 부가적인 기능을 추가하고싶다면 invoke 메소드 내에서 구현을 해주면 된다.

