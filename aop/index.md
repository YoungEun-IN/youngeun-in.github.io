# AOP

 

## AOP  
AOP는 기능을 핵심 비즈니스 로직과 공통 모듈로 구분하고 핵심 로직에 영향을 미치지 않고 사이사이에 공통 모듈을 효과적으로 잘 끼워 넣어 중복성을 감소시킬 수 있는 개발 방법이다.

## AOP 용어 

|용어|설명|
|----|----|
| Advice|`Joinpoint`에서 실행되어야 하는 **프로그램 코드**<br>독립된 클래스의 메소드로 작성함<br>실질적으로 어떤 일을 해야할 지에 대한 것, 실질적인 부가기능을 담은 구현체<br>BEFORE, AROUND, AFTER의 실행 위치 지정|
|Joinpoint|메소드를 호출하는 '시점', 예외가 발생하는 '시점'과 같이 애플리케이션을 실행할 때 **특정 작업이 실행되는 '시점'** 을 의미<br>`Advice`를 적용할 수 있는 후보 지점 혹은 호출 이벤트<br>관심사를 구현한 코드를 끼워 넣을 수 있는 프로그램의 이벤트를 말하며, 예로는 call events, execution events, initialization events 등이 있음|  
|Pointcut|**여러 Joinpoint의 집합체**로 언제 `Advice`를 실행할지를 정의<br>`Target` 클래스와 `Advice`가 결합(`Weaving`)될 때 둘 사이의 결합규칙을 정의| 
|Target|**실질적인 비지니스 로직을 구현하고 있는 코드**<br>`JoinPoint`에서 실제 `Advice`가 적용될 지점|    
|Weaving|**`PointCut`에 `Advice` 메서드가 삽입되는 과정**|

![AOP](https://user-images.githubusercontent.com/50267433/126900960-e0ed4a26-9521-49ac-bfd3-c9a915c12772.png)               

## AOP 구현 
```gradle
implementation 'org.springframework.boot:spring-boot-starter-aop'
```
```java
// @EnableAspectJAutoProxy 생략 가능 
@SpringBootApplication
public class Application {
    public static void main(String[] args) { SpringApplication.run(Application.class,args); }
}
```
```java
@Service
public class AuthServiceImpl {
    public void businessLogicMethod(){
        System.out.println("businessLogicMethod process!");
    }
}
```

### 일반적인 구현      
```java
import org.aspectj.lang.ProceedingJoinPoint;

@Aspect
@Configuration
public class UselessAspect {

    Logger log = LoggerFactory.getLogger(UselessAdvisor.class);

    @Around("execution(* com.demo.study.service.AuthServiceImpl.*(..))")
    public Object stopWatch(ProceedingJoinPoint joinPoint) throws Throwable {
        StopWatch stopWatch = new StopWatch();
        try {
            stopWatch.start();
            return joinPoint.proceed();
        } finally {
            stopWatch.stop();
            log.info("request spent {} ms", stopWatch.getLastTaskTimeMillis());
        }
    }

    /** 
     * @Before
     * @AfterReturning
     * @AfterThrowing
     */
    @After("execution(* com.demo.study.service.AuthServiceImpl.*(..))")
    public void After() throws Throwable {
        log.info("After 어드바이스");
    }

}
```

### 어노테이션으로 구현   
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.CLASS)
public @interface PerformanceCheck {
}
```
```java
@Service
public class AuthServiceImpl {
    @PerformanceCheck
    public void businessLogicMethod(){
        System.out.println("businessLogicMethod process!");
    }
}
```  
```java
import org.aspectj.lang.ProceedingJoinPoint;

@Aspect
@Component
public class UselessAspect {

    Logger log = LoggerFactory.getLogger(UselessAdvisor.class);

    @Around("@annotation(com.demo.study.annotation.PerformanceCheck)")
    public Object stopWatch(ProceedingJoinPoint joinPoint) throws Throwable {
        StopWatch stopWatch = new StopWatch();
        try {
            stopWatch.start();
            return joinPoint.proceed();
        } finally {
            stopWatch.stop();
            log.info("request spent {} ms", stopWatch.getLastTaskTimeMillis());
        }
    }

    /** 
     * @Before
     * @AfterReturning
     * @AfterThrowing
     */
    @After("@annotation(com.demo.study.annotation.PerformanceCheck)")
    public void After() throws Throwable {
        log.info("After 어드바이스");
    }

}
```
* `어노테이션`과 `Aspect`가 동일 위치면 어노테이션만 적어도 된다.   
* 패키지가 다르면 FQCN(Fully qualified Class Name)을 다 입력해주어야 한다.    

### 빈으로 구현   
```java
import org.aspectj.lang.ProceedingJoinPoint;

@Aspect
@Component
public class UselessAspect {

    Logger log = LoggerFactory.getLogger(UselessAdvisor.class);

    @Around("bean(com.demo.study.service.AuthServiceImpl)")
    public Object stopWatch(ProceedingJoinPoint joinPoint) throws Throwable {
        StopWatch stopWatch = new StopWatch();
        try {
            stopWatch.start();
            return joinPoint.proceed();
        } finally {
            stopWatch.stop();
            log.info("request spent {} ms", stopWatch.getLastTaskTimeMillis());
        }
    }

    /** 
     * @Before
     * @AfterReturning
     * @AfterThrowing
     */
    @After("bean(com.demo.study.service.AuthServiceImpl)")
    public void After() throws Throwable {
        log.info("After 어드바이스");
    }

}
```
* `빈`과 `Aspect`가 동일 위치면 어노테이션만 적어도 된다.   
* 패키지가 다르면 FQCN(Fully qualified Class Name)을 다 입력해주어야 한다.    
   
## AOP PointCut 설정  
```java
execution(* com.springbook.biz..*Impl.get*(..))"
```   

* `*` : 리턴값
* `com.springbook.biz..` : 패키지
* `*Impl`: 클래스 이름
* `get*` : 메서드 이름
* `(..)` : 매개변수 
* 중간마다의 `.` : 구분점   

### execution 포인트 컷 리턴

|표현식|설명| 
|----|---|   
|`*`|모든 리턴타입 허용|   
|`void`|리턴타입이 void인 메서드 선택|   
|`!void`|리턴타입이 void가 아닌 메서드 선택|   

### execution 포인트 컷 패키지 

|표현식|설명|  
|----|---|      
|`com.springbook.biz`|정학하게 해당 패키지만 선택|      
|`com.springbook.biz..`|해당 패키지 및 모든 하위 패키지 선택|      
|`com.springbook..impl`|`..`앞 패키지로 시작하면서 마지막 패키지 이름이 `..`뒤로 끝나는 패키지 선택|          
   
### execution 포인트 컷 클래스
|표현식|설명|  
|----|---|      
|`BoardServiceImpl`|정학하게 해당 클래스만 선택|      
|`*Impl`|클래스 이름이 `*`뒷 글자로 끝나는 클래스만 선택|      
|`BoardService+`|해당 클래스는 물론 파생된 모든 자식 클래스도 선택 가능|
|`variable+`|해당 인터페이스를 구현한 모든 클래스 선택 가능 |    

### execution execution 포인트 컷 메서드

|표현식|설명|
|----|---|
|`*(..)`|가장 기본 설정으로 모든 메서드 선택|
|`get*(..)`|메서드 이름이 get으로 시작하는 모든 메서드 선택|
|`매서드이름(..)`|특정 메서드 이름을 가진 메서드 선택|

#### execution 포인트 컷 매개변수 
   
|표현식|설명|
|----|---|
|`(..)`|가장 기본 설정으로서 매개변수 타입의 제한이 없음을 의미|
|`(*)`|반드시 1개의 매개변수를 가지는 메서드만 허용한다는 의미|
|`(com.spring.user.User)`|해당 패키지의 클래스를 가지는 메서드만 허용한다는 의미|
|`(!com.spring.user.User)`|해당 패키지의 클래스를 가지지 않는 메서드만 허용한다는 의미|
|`(Integer, ..)`|한 개 이상의 매개변수를 가지되, 첫 번째 매개변수의 타입이 integer인 메서드만 허용|
|`(Integer, *)`|반드시 두 개의 매개변수를 가지되, 첫 번째 매개변수의 타입이 integer인 메서드만 허용|
        
            
## AOP 구현체  
||SpringAOP|AspectJ|
|-|---------|-------|
|목표|간단한 AOP 제공|완벽한 AOP 제공|
|join point|메서드 레벨만 지원|생성자, 필드, 메서드 등 다양하게 지원|
|weaving|런타임 시에만 가능|런타임은 제공하지 않음, compile-time, post-compile, load-time 제공|   
|대상|Spring Container가 관리하는 Bean에만 가능|모든 JAVA Object에 가능|  

## 참고
https://github.com/kwj1270/TIL_Seminar/blob/master/AOP.md

https://swingswing.tistory.com/m/272

