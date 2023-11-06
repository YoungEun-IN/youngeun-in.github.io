# Spring PSA


PSA란 환경의 변화와 관계없이 일관된 방식의 기술로의 접근 환경을 제공하는 추상화 구조를 말한다. POJO원칙을 철저히 따른 Spring의 기능으로 Spring에서 동작할 수 있는 Library들은 POJO원칙을 지키게끔 PSA형태의 추상화가 되어있음을 의미한다.

추상화 계층을 사용하여 어떤 기술을 내부에 숨기고 개발자에게 편의성을 제공해주는 것이 서비스 추상화(Service Abstraction)이다. **하나의 추상화로 여러 서비스를 묶어둔 것을 Spring에서 Portable Service Abstraction이라고 한다.**
 
 ## PSA 예시

 ### Spring Web MVC

원래 Servlet을 사용하려면 HttpServlet을 상속받은 클래스를 만들고 Get, Post 등에 대한 메소드를 오버라이딩하여 사용해야 한다.

@Controller에 대해 생각해보자. 클래스 선언부에 이 어노테이션을 사용하면 클래스 내부에서 메소드에 @GetMapping, @PostMapping등을 사용할 수 있고 클래스 선언부에 @RequestMapping을 사용할 수 있다. 이것이 "잘 만든 인터페이스"의 예시이다. 이러한 편의성 제공이 서비스 추상화의 목적이다. 우리는 @Controller, @GetMapping같은 기능들의 뒷단의 복잡한 코드에 대해서 깊게 고려하지 않아도 되고 이런 기술들을 기반으로 하여 기존 코드를 거의 변경하지 않아도 된다.

또한, 실행 서버를 Tomcat이 아닌 netty로 변경하고 싶다면 프로젝트 설정에 spring-boot-starter-web 의존성 대신 spring-boot-starter-webflux 의존성을 받도록 하면 된다.

여러가지 복잡하고 잘 만들어진 인터페이스들, 의존성같은 기술들을 기반으로 하여 Spring은 개발자에게 편리한 환경을 제공한다.

 ### Spring Transaction

JDBC를 사용해서 DB접근하고 작업, 트랜잭션을 구현할 때는 연결, 작업할 쿼리문, commit, rollback하는 코드를 전부 작성해야 했다. 그러나 Spring에선 메소드에 @Transactional을 통해 트랜잭션 처리를 간단하게 할 수 있다.

## 참고

https://velog.io/@kimsunfang/Spring%EC%9D%98-3%EB%8C%80-%EC%9A%94%EC%86%8C

