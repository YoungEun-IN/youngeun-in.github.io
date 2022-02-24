---
title: AOP
date: 2022-02-24T15:12:26+09:00
categories:
  - spring
tags: 
  - spring-aop
  - dynamic-proxy
--- 

## 📕 AOP  
> Aspect-Oriented Programming | 관점 지향 프로그래밍   
   
* OOP 객체지향 프로그램   
* AOP 관점지향 프로그램    
       
AOP는 OOP를 더욱 OOP답게 프로그래밍 할 수 있게 도와주는 것으로      
**애플리케이션의 `핵심적인 기능`과 `부가적인 기능`을 `분리`해**    
**`Aspect`라는 모듈로 만들어 설계하고 개발하는 방법이다.**        

실제로 Spring doc document 에서는 아래와 같이 말하고 있다.  
```   
Aspect-oriented Programming (AOP) complements    
Object-oriented Programming (OOP) by providing   
another way of thinking about program structure   

AOP는 프로그램 구조에 대한 다른 생각의 방향을 제공해주면서 OOP를 보완하고 있다.       
```      
  
예를 들면 어떠한 클래스를 대상으로 `핵심 기능`과 `부가적인 기능`의 관점으로         
**공통 사용 부가 기능들을 외부의 독립된 클래스로 분리하고**                
**이를 모듈화하여 재사용할 수 있게끔 하는 프로그래밍 기법이다.**                 
                 
**AOP 구현체**    
* AspectJ : 다양한 포인트 컷 제공       
* 스프링 AOP : 제한적이지만 효율적인 기능 제공   

||SpringAOP|AspectJ|
|-|---------|-------|
|목표|간단한 AOP 제공|완벽한 AOP 제공|
|join point|메서들 레벨만 지원|생성자, 필드, 메서드 등 다양하게 지원|
|weaving|런타임 시에만 가능|런타임은 제공하지 않음, compile-time, post-compile, load-time 제공|   
|대상|Spring Container가 관리하는 Bean에만 가능|모든 JAVA Object에 가능|  

## 📘 AOP 용어 

|용어|설명|
|----|----|
|**Aspect(Advisor)**|**✔ 특정 한 가지 기능에 대해 흩어진 관심사를 모듈화하여 묶은 클래스**<br>- `PointCut + Advice`를 묶은 메서드를 정의 및 관리한다.|
|Target|**✔ 비즈니스(핵심) 로직을 구현하는 AOP의 대상 '클래스'**|  
|Advice|**✔ AOP 메서드 : 부가 기능을 적용시키는 메서드(부가 기능 로직 메서드)**<br>**적용 시점도 지정할 수 있다.**<br>- before : 비즈니스 메소드 실행 전에 동작<br>- after : 비즈니스 메소드 실행 후에 동작<br>- after-returning : 비즈니스 메소드 실행 중 리턴 되는 순간<br>- after-throwing : 비즈니스 메소드 실행 중 에러 발생 순간<br>- around	: 비즈니스 메소드 실행 전/후에 동작| 
|JoinPoint|**✔ AOP가 적용될 수 있는 `메소드`, 필드, 객체, 생성자 등의 요소**<br>- 타겟 클래스의 요소들을 의미하며 **PointCut의 후보**<br>- **Spring AOP에서는 메서드만 지정 가능**|
|PointCut|**✔ AOP 적용 가능 요소들 중에서 '실제 적용될 요소들'을 의미한다.**<br>- `JoinPoint`에서 실제 `Advice`가 적용될 지점<br>- **Spring AOP 에서는 advice가 적용될 메서드를 의미**|    
|Weaving|**✔  PointCut이 호출될 때 Advice가 호출되는 과정을 의미한다.**<br>즉, **PointCut에 Advice 메서드가 삽입되는 과정을 의미한다.**<br>- 컴파일 전에는 핵심 로직과 인프라 로직이 분리되어있지만<br>- 컴파일타임에 바이트 코드를 넣는다던지 런타임에 프록시를 만들던지<br>- 관심사를 분리한 코드를 비즈니스 코드와 병합하는 작업을 의미한다.<br>**Weaving 처리 방식 (AOP 구현 방법)**<br>- 컴파일 타임 위빙 : `a.java -> a.clss` 컴파일 될 때<br>- 로딩 타임 위빙 : 클래스파일을 클래스 로더가 메모리에 로드할 때<br>- 런타임/프록시 위빙 : 타겟 클래스에 부가 기능을 추가하도록 Proxy 객체를 만들어(감싸서) 실행<br>- 스프링 AOP는 **런타임 위빙**만을 지원한다.(IOC/DI를 이용한 방법)|    
|proxy|**✔ Target 객체에 Advice를 적용시키는 래퍼 클래스**|    

![AOP](https://user-images.githubusercontent.com/50267433/126900960-e0ed4a26-9521-49ac-bfd3-c9a915c12772.png)    
        
1. 애플리케이션은 자연스럽게 여러 메서드(Join Point)를 호출한다.      
2. 이때 특정 **PointCut으로 지정한 메소드가 호출되는 순간**       
3. **Aspect 의 Advice 메소드가 호출된다.**             
4. Advice 메소드의 **동작 시점은 5가지로 정할 수 있다.**               
5. 애스팩트 설정에 따라 **위빙 처리되어 프록시 객체가 생성된다.(동적 프록시 생성)**                 
6. 프록시 객체를 통해 부가기능이 포함된 비즈니스 로직을 수행한다.              

### 📖 AOP 구현 
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

#### 📄 일반적인 구현      
```java
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

#### 📄 어노테이션으로 구현   
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

#### 📄 빈으로 구현   
```java
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
   
### 📖 AOP PointCut 설정  
```java
execution(* com.springbook.biz..*Impl.get*(..))"
```   

* `*` : 리턴값
* `com.springbook.biz..` : 패키지
* `*Impl`: 클래스 이름
* `get*` : 메서드 이름
* `(..)` : 매개변수 
* 중간마다의 `.` : 구분점   

#### 📄 execution 포인트 컷 리턴

|표현식|설명| 
|----|---|   
|`*`|모든 리턴타입 허용|   
|`void`|리턴타입이 void인 메서드 선택|   
|`!void`|리턴타입이 void가 아닌 메서드 선택|   

#### 📄 execution 포인트 컷 패키지 

|표현식|설명|  
|----|---|      
|`com.springbook.biz`|정학하게 해당 패키지만 선택|      
|`com.springbook.biz..`|해당 패키지 및 모든 하위 패키지 선택|      
|`com.springbook..impl`|`..`앞 패키지로 시작하면서 마지막 패키지 이름이 `..`뒤로 끝나는 패키지 선택|          
   
#### 📄 execution 포인트 컷 클래스
|표현식|설명|  
|----|---|      
|`BoardServiceImpl`|정학하게 해당 클래스만 선택|      
|`*Impl`|클래스 이름이 `*`뒷 글자로 끝나는 클래스만 선택|      
|`BoardService+`|해당 클래스는 물론 파생된 모든 자식 클래스도 선택 가능|
|`variable+`|해당 인터페이스를 구현한 모든 클래스 선택 가능 |    

#### 📄 execution execution 포인트 컷 메서드

|표현식|설명|
|----|---|
|`*(..)`|가장 기본 설정으로 모든 메서드 선택|
|`get*(..)`|메서드 이름이 get으로 시작하는 모든 메서드 선택|
|`매서드이름(..)`|특정 메서드 이름을 가진 메서드 선택|

#### 📄 execution 포인트 컷 매개변수 
   
|표현식|설명|
|----|---|
|`(..)`|가장 기본 설정으로서 매개변수 타입의 제한이 없음을 의미|
|`(*)`|반드시 1개의 매개변수를 가지는 메서드만 허용한다는 의미|
|`(com.spring.user.User)`|해당 패키지의 클래스를 가지는 메서드만 허용한다는 의미|
|`(!com.spring.user.User)`|해당 패키지의 클래스를 가지지 않는 메서드만 허용한다는 의미|
|`(Integer, ..)`|한 개 이상의 매개변수를 가지되, 첫 번째 매개변수의 타입이 integer인 메서드만 허용|
|`(Integer, *)`|반드시 두 개의 매개변수를 가지되, 첫 번째 매개변수의 타입이 integer인 메서드만 허용|
        

## 📗 스프링 AOP  
스프링 AOP 는 **런타임 위빙을 지원하기에 Proxy 기반의 AOP 구현 기능을 지원한다.**             
이러한 **Proxy 기반의 AOP는 스프링 빈에만 적용가능하기에 빈으로 등록된 대상만 AOP 대상이 된다.**         
또한, **스프링 AOP 특징상 메서드만을 지원할 수 있다.**        
           
**프록시 패턴**   
```java
@Primary
@Service
public class ProxySimpleEventService implements EventService{

    private final EventService simpleEventService;

    ProxySimpleEventService(EventService simpleEventService) {
        this.simpleEventService = simpleEventService;
    }

    @Override
    public void createEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.createEvent();
        System.out.println("createEvent 실행시간: "+(System.currentTimeMillis()-begin));
    }

    @Override
    public void publishEvent() {
        long begin = System.currentTimeMillis();
        simpleEventService.publishEvent();
        System.out.println("createEvent 실행시간: "+(System.currentTimeMillis()-begin));
    }

    @Override
    public void deleteEvent() {
        simpleEventService.deleteEvent();
    }
}
```

프록시 패턴은 **AOP에서 기존 코드의 변경 없이 접근 제어 또는 부가 기능 추가하기 위해 사용된다.**              
그러나 일반적인 방법으로 프록시 패턴을 구현하면 아래와 같은 문제점이 발생한다.         
                
**문제점**  
* 매번 Proxy 클래스를 작성해야 한다.    
* 하나의 클래스가 아닌 여러 클래스를 대상으로 적용하기 힘들다.    
* 객체 관계가 복잡해지고 관리하기 어렵다.     
         
이러한 문제점을 해결하기 위해 등장한 것이 바로 `Spring AOP`와 `Dynamic Proxy`다.  
**스프링 IoC 컨테이너가 제공하는 기반 시설과 Dynamic Proxy를 사용하여 여러 복잡한 문제 해결했다.**      
  
* **Dynamic Proxy :** 동적으로 프록시 객체 생성하는 방법     
    **JDK Dynamic Proxy :** 자바가 제공하는 인터페이스 기반 프록시 생성 라이브러리      
    * **CGlib :** 자바가 제공하는 클래스 기반 프록시 생성 라이브러리    
* **Spring IoC :** **기존 빈을 대체하는 '동적 프록시 빈'을 만들어 등록한다.**      
    * 클라이언트 코드 변경이 없다.      
    * `AbstractAutoProxyCreator implements BeanPostProcessor`를 기반으로 만든다.    
    * 즉 빈이 생성된 후, 프록시를 생성하는 로직을 수행한다.       
  
이렇듯 **런타임시에 동적으로 프록시 빈을 생성하고 AOP를 적용시키는 것을 런타임 위빙이라 부른다.**       
   
### 🔍 런타임 위빙(프록시 위빙)      
런타임 위빙은 **런타임에 동적으로 Proxy를 생성하여 실제 Target 객체의 변형없이 AOP를 수행하는 것을 의미한다.**      
스프링을 기준으로 말하자면 메서드만이 JoinPoint가 될 수 있기에 Method 호출 시에 위빙이 이루어 지는 방식이다.        
   
런타임 위빙 또한, 일반적인 Proxy 패턴을 개선한 Dynamic Proxy를 사용하고 있지만 아래와 같은 문제가 있다.     
* 포인트 컷에 대한 어드바이스 적용 갯수가 늘어 날수록 성능이 떨어진다.            
* 메소드 호출에 대해서만 어드바이스를 적용 할 수 있다.(프록시이기에)                 
   
**AOP 프록시 객체**
```
org.woowacourse.aoppractice.service.AuthServiceImpl$$EnhancerBySpringCGLIB$$dbdb402d
```

* 기존 객체 : org.woowacourse.aoppractice.service.AuthServiceImpl      
* 프록시 객체 : org.woowacourse.aoppractice.service.AuthServiceImpl$$EnhancerBySpringCGLIB$$dbdb402d    

위 코드를 보면, Target 클래스 자체를 프록시로 감싸는 것을 알 수 있다.   
      
<img width="842" alt="aop-proxy" src="https://user-images.githubusercontent.com/50267433/126906051-ec8d6b7c-0f34-44c7-8a8a-51523a4c6a84.png">

프록시 위빙이 가능한 이유는 **상속을 이용한 방식이기에 다형성을 적용시킬 수 있다.**    
즉, 위 그림에서 `AuthController`가 `AuthService`의 프록시 클래스인 `AuthService$$블라블라`를 참조할 수 있던 것이다.      
            
그런데 한가지 생각해볼 점이 있다.           
**DynamicProxy는 상속/구현을 적용하니 아래와 같은 요소들은 가능할까? 🤔**          
     
* final 클래스   
* final 메서드   
* private 메서드   
   
당연하게도, **자바 문법적으로 상속 및 오버라이딩을 지원하지 못하므로 적용이 되지 않는다.**                   
이와 비슷하게 Spring에서 지원해주는 `@Trancsactional` 어노테이션이 있는데               
`@Trancsactional` 어노테이션 또한 AOP기반이기에 private 메서드를 붙이면 작동을 하지 않는다.         
     
**@Trancsactional**       
* 로직 시작시 트랜잭션을 열어줌    
* 로직 끝날시 commit하고 트랜잭션을 닫아줌        
* 트랜잭션에 관련된 인프라 로직을 지원하기에 우리는 비즈니스 로직에 집중할 수 있게해준다.                  
            
# 참고            
[백기선-스프링 프레임워크 핵심 기술](https://www.inflearn.com/course/spring-framework_core)      
[스프링 퀵 스타트](http://www.yes24.com/Product/Goods/29173715)      
[테코톡-스프링 AOP](https://www.youtube.com/watch?v=Hm0w_9ngDpM)      
[장인 개발자를 꿈꾸는 : 기록하는 공간](https://devbox.tistory.com/entry/spring-AOP-%EC%9A%A9%EC%96%B4-%EC%84%A4%EB%AA%85)           
[Carrey's 기술블로그](https://jaehun2841.github.io/2018/07/22/2018-07-22-spring-aop4/#joinpoint-%EA%B4%80%EB%A0%A8-annotations)          
[진짜 개발자](https://galid1.tistory.com/498)       
[기억보단 기록을](https://jojoldu.tistory.com/69)         
[리다양의 개발 세상](https://onlyformylittlefox.tistory.com/16)   
[Carrey's 기술블로그-AspectJ 나중에 참고하면 좋음](https://jaehun2841.github.io/2018/07/22/2018-07-22-spring-aop4/#aspectj%EB%9E%80)    
[논리적 코딩](https://logical-code.tistory.com/118)             
     
