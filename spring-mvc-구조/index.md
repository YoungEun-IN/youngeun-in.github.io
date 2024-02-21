# Spring MVC 구조


## DispatcherServlet 구조

스프링 MVC는 프론트 컨트롤러 패턴으로 구현되어 있다.
**스프링 MVC의 프론트 컨트롤러가 디스패처 서블릿(DispatcherServlet)이다.**

FrontController 패턴 특징은 다음과 같다.
- 프론트 컨트롤러 서블릿 하나로 클라이언트의 요청을 받음
- 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호
- 입구를 하나로 하여 공통 처리 가능
- 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 됨

DispacherServlet는 부모 클래스에서 HttpServlet 을 상속 받아서 사용하고, 서블릿으로 동작한다. 

상속관계는 다음과 같다. DispatcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet

요청흐름은 다음과 같다.
1. 서블릿이 호출되면 HttpServlet 이 제공하는 serivce() 가 호출된다.
2. 스프링 MVC는 DispatcherServlet 의 부모인 FrameworkServlet 에서 service() 를 오버라이드 해두었다.
3. FrameworkServlet.service() 를 시작으로 여러 메서드가 호출되면서 DispacherServlet.doDispatch() 가 호출된다.

스프링 부트는 DispacherServlet 을 서블릿으로 자동으로 등록하면서 모든 경로( urlPatterns="/" )에 대해서 매핑한다. 더 자세한 경로가 우선순위가 높으므로 기존에 등록한 서블릿도 함께 동작한다.

## 요청 흐름

<img width="791" alt="스크린샷 2024-02-19 오후 5 50 31" src="https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/d820f116-ab08-448f-9856-24f026ad63de">

1. 핸들러 조회: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
2. 핸들러 어댑터 조회: 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
3. 핸들러 어댑터 실행: 핸들러 어댑터를 실행한다.
4. 핸들러 실행: 핸들러 어댑터가 실제 핸들러를 실행한다.
5. ModelAndView 반환: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다.
6. viewResolver 호출: 뷰 리졸버를 찾고 실행한다.
   - JSP의 경우: InternalResourceViewResolver 가 자동 등록되고, 사용된다.
8. View반환: 뷰리졸버는 뷰의 논리이름을 물리이름으로 바꾸고, 렌더링 역할을 담당하는 뷰객체를 반환한다.
   - JSP의 경우 InternalResourceView(JstlView) 를 반환하는데, 내부에 forward() 로직이 있다.
9. 뷰렌더링: 뷰를 통해서 뷰를 렌더링한다.

## 참고
https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1

