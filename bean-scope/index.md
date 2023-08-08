# Bean Scope


Bean은 스프링에서 사용하는 POJO 기반 객체다. 상황과 필요에 따라 Bean을 사용할 때 하나만 만들어야 할 수도 있고, 여러개가 필요할 때도 있고, 어떤 한 시점에서만 사용해야할 때가 있을 수 있다. 이를 위해 **Scope를 설정해서 Bean의 사용 범위를 개발자가 설정할 수 있다.**

우선 **따로 설정을 해주지 않으면, Spring에서 Bean은 `Singleton`으로 생성된다.** 특정 타입의 Bean을 딱 하나만 만들고 모두 공유해서 사용하기 위함이다. 보통은 Bean을 이렇게 하나만 만들어 사용하는 경우가 대부분이지만, 요구사항이나 구현에 따라 아닐 수도 있을 것이다. 따라서 Bean Scope는 싱글톤 말고도 여러가지를 지원해준다.

## Scope 종류

- singleton
  - 기본 스코프, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프이다.
- prototype
  - 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프이다.
- 웹 관련 스코프
  - request
    - 웹 요청이 들어오고 나갈때 까지 유지되는 스코프이다.
  - session
    - 웹 세션이 생성되고 종료될 때 까지 유지되는 스코프이다.
  - application
    - 웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프이다.

## Scope 설정

Scope들은 Bean으로 등록하는 클래스에 어노테이션으로 설정해줄 수 있다.

```java
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Service;
 
@Scope("prototype")
@Component
public class UserController {
}
```

## 프로토타입 스코프
프로토타입 빈의 특징을 정리하면 다음과 같다.

- 스프링 컨테이너에 요청할 때 마다 새로 생성된다.
- 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입 그리고 초기화까지만 관여한다.
- 종료 메서드가 호출되지 않는다. 그래서 프로토타입 빈은 프로토타입 빈을 조회한 클라이언트가 관리해야 한다. 종료 메서드에 대한 호출도 클라이언트가 직접 해야한다.

## 참고
https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8

