---
title: Filter vs Interceptor vs AOP
date: 2022-03-07T15:12:26+09:00
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

* __init():__ 필터 인스턴스 초기화
* __doFilter():__ 전/후 처리
* __destroy():__ 필터 인스턴스 종료

## Interceptor
인터셉터는 스프링의 DistpatcherServlet이 컨트롤러를 호출하기 전, 후로 끼어들기 때문에 **스프링 컨텍스트(Context, 영역) 내부에서 Controller(Handler)에 관한 요청과 응답을 처리한다.**
스프링의 모든 빈 객체에 접근할 수 있다. 인터셉터는 여러 개를 사용할 수 있고 로그인 체크, 권한 체크, 프로그램 실행 시간 계산작업 로그 확인 등의 업무처리에 사용한다.

* __preHandler():__ 컨트롤러 메소드가 실행되기 전
* __postHanler():__ 컨트롤러 메소드 실행직 후 view 페이지 렌더링 되기 전
* __afterCompletion():__ view 페이지가 렌더링 되고 난 후

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
