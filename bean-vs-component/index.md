# Bean vs Component


## @Bean
@Bean은 **메소드 레벨에서 선언**하며, 반환되는 객체(인스턴스)를 개발자가 수동으로 빈으로 등록하는 애노테이션이다.

```java
@Configuration
public class AppConfig {
   @Bean
   public MemberService memberService() {
      return new MemberServiceImpl();
   }
}
```

## @Component
@Component는 **클래스 레벨에서 선언**함으로써 스프링이 런타임시에 컴포넌트스캔을 하여 자동으로 빈을 찾고(detect) 등록하는 애노테이션이다.

```java
@Component
public class Utility {
   // ...
}
```

## @Bean vs @Component
**@Bean의 경우 개발자가 컨트롤이 불가능한 외부 라이브러리들을 Bean으로 등록하고 싶은 경우에 사용된다.**
(예를 들면 ObjectMapper의 경우 ObjectMapper Class에 @Component를 선언할수는 없으니 ObjectMapper의 인스턴스를 생성하는 메소드를 만들고 해당 메소드에 @Bean을 선언하여 Bean으로 등록한다.)

반대로 **개발자가 직접 컨트롤이 가능한 Class들의 경우엔 @Component를 사용한다.**

@Bean과 @Component는 각자 선언할 수 있는 타입이 정해져있어 해당 용도외에는 컴파일 에러를 발생시킨다.

## 참고
https://youngjinmo.github.io/2021/06/bean-component/

https://jojoldu.tistory.com/27

