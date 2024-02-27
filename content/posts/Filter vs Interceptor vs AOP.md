---
title: Filter vs Interceptor vs AOP
date: 2023-03-07T15:12:26+09:00
categories:
  - spring
tags: 
  - filter
  - interceptor
  - aop
---

## Filter
Filter(필터)는 요청과 응답을 거른 뒤 정제하는 역할을 한다.
서블릿 필터는 DispatcherServlet 이전에 실행이 되는데 필터가 동작하도록 지정된 자원의 앞단에서 요청내용을 변경하거나, 여러 가지 체크를 수행할 수 있다.
또한 자원의 처리가 끝난 후 응답 내용에 대해서도 변경하는 처리를 할 수가 있다.
즉 **스프링 컨텍스트 외부에 존재하여 스프링과 무관한 자원에 대해 동작한다.**
보통 web.xml에 등록하고, 일반적으로 인코딩 변환 처리, XSS 방어 등의 요청에 대한 처리로 사용된다.

### 필터 흐름

```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
```

### 필터 인터페이스

```java
public interface Filter {
  public default void init(FilterConfig filterConfig) throws ServletException{}
  public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException;
  public default void destroy() {}
}
```

필터 인터페이스를 구현하고 등록하면 서블릿 컨테이너가 필터를 싱글톤 객체로 생성하고, 관리한다.
- `init():` 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출된다.
- `doFilter():` 고객의 요청이 올 때 마다 해당 메서드가 호출된다. 필터의 로직을 구현하면 된다.
- `destroy():` 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출된다.

## Interceptor
인터셉터는 스프링의 DistpatcherServlet이 컨트롤러를 호출하기 전, 후로 끼어들기 때문에 **스프링 컨텍스트(Context, 영역) 내부에서 Controller(Handler)에 관한 요청과 응답을 처리한다.**
스프링의 모든 빈 객체에 접근할 수 있다. 인터셉터는 여러 개를 사용할 수 있고 로그인 체크, 권한 체크, 프로그램 실행 시간 계산작업 로그 확인 등의 업무처리에 사용한다.

### 스프링 인터셉터 흐름

```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러
```

- 스프링 인터셉터는 디스패처 서블릿과 컨트롤러 사이에서 컨트롤러 호출 직전에 호출 된다.
- 스프링 인터셉터는 스프링 MVC가 제공하는 기능이기 때문에 결국 디스패처 서블릿 이후에 등장하게 된다.
- 스프링 인터셉터에도 URL 패턴을 적용할 수 있는데, 서블릿 URL 패턴과는 다르고, 매우 정밀하게 설정할 수 있다.

## 스프링 인터셉터 인터페이스

스프링의 인터셉터를 사용하려면 `HandlerInterceptor` 인터페이스를 구현하면 된다.
```java
public interface HandlerInterceptor {
  default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {}
  default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {}
  default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {}
}
```

- 서블릿 필터의 경우 단순하게 `doFilter()` 하나만 제공된다. 인터셉터는 컨트롤러 호출 전( `preHandle` ), 호출 후( `postHandle` ), 요청 완료 이후( `afterCompletion` )와 같이 단계적으로 세분화 되어 있다.
- 서블릿 필터의 경우 단순히 `request` , `response` 만 제공하지만, 인터셉터는 어떤 컨트롤러( `handler` )가 호출되는지 호출 정보도 받을 수 있다. 그리고 어떤 `modelAndView` 가 반환되는지 응답 정보도 받을 수 있다.

### 스프링 인터셉터 호출 흐름

#### 정상 흐름
<img width="797" alt="스크린샷 2024-02-22 오후 7 35 59" src="https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/11e8dcd1-88ed-478c-9fcd-c76ff4e7feb3">

- preHandle` : 컨트롤러 호출 전에 호출된다. (더 정확히는 핸들러 어댑터 호출 전에 호출된다.)
  - `preHandle` 의 응답값이 `true` 이면 다음으로 진행하고, `false` 이면 더는 진행하지 않는다. `false` 인 경우 나머지 인터셉터는 물론이고, 핸들러 어댑터도 호출되지 않는다. 그림에서 1번에서 끝이 나버린다.
- `postHandle` : 컨트롤러 호출 후에 호출된다. (더 정확히는 핸들러 어댑터 호출 후에 호출된다.)
- `afterCompletion` : 뷰가 렌더링 된 이후에 호출된다.

#### 예외 상황
<img width="797" alt="스크린샷 2024-02-22 오후 7 37 48" src="https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/90eaeb1e-45ed-4722-9ff0-8154666df435">

- `preHandle` : 컨트롤러 호출 전에 호출된다.
- `postHandle` : 컨트롤러에서 예외가 발생하면 `postHandle` 은 호출되지 않는다.
- `afterCompletion` : `afterCompletion` 은 항상 호출된다. 이 경우 예외( `ex` )를 파라미터로 받아서 어떤 예외가 발생했는지 로그로 출력할 수 있다.

## AOP
AOP는 OOP를 보완하기 위해 나온 개념이다. 객체 지향의 프로그래밍을 했을 때 중복을 줄일 수 없는 부분을 줄이기 위해 종단면(관점)에서 바라보고 처리한다.
주로 **'로깅', '트랜잭션', '에러 처리' 등 비즈니스단의 메소드에서 조금 더 세밀하게 조정할 때 사용한다.**
Interceptor나 Filter와는 달리 메소드 전후의 지점에 자유롭게 설정이 가능하다. Interceptor와 Filter는 주소로 대상을 구분해서 걸러내야 하는 반면, AOP는 주소, 파라미터, 애노테이션 등 다양한 방법으로 대상을 지정할 수 있다.

* __@Before:__ 대상 메소드의 수행 전
* __@After:__ 대상 메소드의 수행 후
* __@After-returning:__ 대상 메소드의 정상적인 수행 후
* __@After-throwing:__ 예외발생 후
* __@Around:__ 대상 메소드의 수행 전, 후

## 참고
https://dejavuhyo.github.io/posts/spring-filter-interceptor-aop-differences/

https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2
