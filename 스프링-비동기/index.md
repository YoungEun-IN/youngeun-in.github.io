# 스프링 비동기


외부 API에 작업 요청하고 외부 API서버에서 요청을 처리하는데 오랜시간이 걸리는 경우 비동기 방식으로 처리하면 효율적이다.

## @EnableAsync

**@Async 는 비동기적으로 처리를 할 수 있게끔 스프링에서 제공하는 어노테이션이다.** 해당 어노테이션을 붙이게 되면 각기 다른 쓰레드로 실행이 된다. 즉, 호출자는 해당 메서드가 완료되는 것을 기다릴 필요가 없다.

이 어노테이션을 사용하기 위해서는 @EnableAsync 가 달려있는 configuration 클래스가 우선적으로 필요하다. @EnableAsync 는 스프링의 @Async 어노테이션을 감지한다.

```java
@Configuration
@EnableAsync
public class AsyncConfig {
}
```

@Async 어노테이션을 사용하기 위해서는 2가지 제약조건이 있다.
- **public 메서드일 것**
  - 프록시를 사용하기 위해서 메서드는 public이어야 한다.
- **동일 클래스에서 호출하는 Self-invocation 이어서는 안된다는 것**
  - Self-invocation를 사용하게되면 프록시를 무시하고 바로 메서드를 호출한다.

기본적으로, 스프링은 비동기적으로 메서드를 실행하기 위해서 `SimpleAsyncTaskExecutor`를 사용한다.
SimpleAsyncTaskExecutor는 요청이 오는대로 계속해서 쓰레드를 생성하는 방식이다. 이것을 어플리케이션 레벨 또는 각 메서드 레벨에서 override 함으로써 default를 변경할 수 있다.

### 메서드 레벨에서 Override

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

## 참고
https://bepoz-study-diary.tistory.com/399

