# 스프링 비동기


## 스프링 비동기 설정

**@Async는 비동기적으로 처리를 할 수 있게끔 스프링에서 제공하는 어노테이션이다.** 해당 어노테이션을 붙이게 되면 각기 다른 쓰레드로 실행이 된다. 즉, 호출자는 해당 메서드가 완료되는 것을 기다릴 필요가 없다.

이 어노테이션을 사용하기 위해서는 @EnableAsync 가 달려있는 configuration 클래스가 우선적으로 필요하다. **@EnableAsync는 스프링의 @Async 어노테이션을 감지한다.**

```java
@Configuration
@EnableAsync
public class AsyncConfig {
}
```

기본적으로, 스프링은 비동기적으로 메서드를 실행하기 위해서 `SimpleAsyncTaskExecutor`를 사용한다.
SimpleAsyncTaskExecutor는 요청이 오는대로 계속해서 쓰레드를 생성하는 방식이다. 이것을 어플리케이션 레벨 또는 각 메서드 레벨에서 override 함으로써 default를 변경할 수 있다.

### 메서드 레벨에서 Override

설정 클래스에서 필요한 실행자를 선언해주어야한다. 그 후 실행자 이름을 @Async에서 속성값으로 제공해주어야 한다.

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean("customAsyncExecutor")
    public Executor customAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(5);
        executor.setThreadNamePrefix("bepoz");
        executor.initialize(); // 꼭 써줘야 한다.
        return executor;
    }
}
```

```java
@Service
@Slf4j
public class AsyncService {
    @Async("customAsyncExecutor") // @Async value로 등록한 Executor의 이름을 입력
    public void call() {
        log.info("async Test");
    }
}
```

### 어플리케이션 레벨에서 Override

설정 클래스는 AsyncConfigurer 인터페이스를 구현해주어야한다. 이는 getAsyncExecutor() 메소드를 구현해야한다는 의미이다.

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(5);
        executor.setThreadNamePrefix("bepoz");
        executor.initialize(); // 꼭 써줘야 한다.
        return executor;
    }
}
```

```java
@Service
@Slf4j
public class AsyncService {
    @Async
    public void call() {
        log.info("async Test");
    }
}
```

## 스프링 비동기 사용

@Async 어노테이션을 사용하기 위해서는 2가지 제약조건이 있다.
- **public 메서드일 것**
  - 프록시를 사용하기 위해서 메서드는 public이어야 한다.
- **동일 클래스에서 호출하는 Self-invocation 이어서는 안된다는 것**
  - Self-invocation를 사용하게되면 프록시를 무시하고 바로 메서드를 호출한다.

### 리턴타입이 없는 메소드

다음과 같이 간단한 설정으로 리턴타입이 void인 메소드가 비동기로 작동한다.

```java
    @Async
    public void asyncMethodWithVoidReturnType(){
        System.out.println("Execute method asynchronously. "
            +Thread.currentThread().getName());
    }
```

### 리턴타입이 있는 메소드

Future 객체에 실제 리턴값을 넣음으로서 @Async 는 리턴타입이 있는 메소드에 적용할 수 있다.

```java
    @Async
    public Future<String> asyncMethodWithReturnType() {
        System.out.println("Execute method asynchronously - " + Thread.currentThread().getName());
        try {
            Thread.sleep(5000);
            return new AsyncResult<String>("hello world !!!!");
        } catch (InterruptedException e) {//}
            return null;
        }
    }
```

## 예외 처리

메소드의 리턴타입이 Future일 경우 예외처리는 쉽다. Future.get() 메소드가 예외를 발생시킨다. 하지만 리턴타입이 void일 때, 예외는 호출 스레드에 전달되지 않을 것이다. 따라서 예외 처리를 위한 추가 설정이 필요하다.

AsyncUncaughtExceptionHandler 인터페이스를 구현함으로서 커스텀 비동기 예외처리기를 만들 수 있다. handleUncaughtException() 메소드는 잡히지않은 uncaught 비동기 예외가 발생할때 호출된다.

```java
    @Override
    public void handleUncaughtException(Throwable throwable, Method method, Object... obj) {
        System.out.println("Exception message - " + throwable.getMessage());
        System.out.println("Method name - " + method.getName());
        for (Object param : obj) {
            System.out.println("Parameter value - " + param);
        }
    }
```

이후 설정 클래스에 의해 구현된 AsyncConfigurer 인터페이스를에 커스텀 비동기 예외처리기를 리턴하는 getAsyncUncaughtExceptionHandler() 메소드를 오버라이드하여 추가 한다.

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(5);
        executor.setThreadNamePrefix("bepoz");
        executor.initialize(); // 꼭 써줘야 한다.
        return executor;
    }
}
```

## 참고
https://bepoz-study-diary.tistory.com/399

https://springboot.tistory.com/38

