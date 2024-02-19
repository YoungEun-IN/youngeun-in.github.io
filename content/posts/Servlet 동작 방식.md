---
title: Servlet 동작 방식
date: 2022-02-23T15:12:26+09:00
categories:
  - java
tags: 
  - servlet-container
---
## 서블릿 컨테이너
`서블릿 컨테이너`는 서블릿을 담아 관리한다. 서블릿은 싱글톤으로 관리된다.

![image](https://user-images.githubusercontent.com/46465928/155554827-4e08d4c2-9239-4ec2-8522-c6c0b7ed0dac.png)

## 웹 애플리케이션 서버의 요청 응답 구조
<img width="1000" alt="스크린샷 2024-02-19 오후 4 44 15" src="https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/85f8fd97-d891-400a-ad4a-1b8f4eefe42c">

### HttpServletRequest 역할
HTTP 요청 메시지를 개발자가 직접 파싱해서 사용해도 되지만, 매우 불편할 것이다. 서블릿은 개발자가 HTTP 요청 메시지를 편리하게 사용할 수 있도록 개발자 대신에 HTTP 요청 메시지를 파싱한다. 그리고 그 결과를 `HttpServletRequest` 객체에 담아서 제공한다.

HttpServletRequest를 사용하면 다음과 같은 HTTP 요청 메시지를 편리하게 조회할 수 있다.

```
  POST /save HTTP/1.1
  Host: localhost:8080
  Content-Type: application/x-www-form-urlencoded
  username=kim&age=20
```

- START LINE
  - HTTP 메소드
  - URL
  - 쿼리 스트링
  - 스키마, 프로토콜
- 헤더
  - 헤더 조회
- 바디
  - form 파라미터 형식 조회
  - message body 데이터 직접 조회


HttpServletRequest 객체는 추가로 여러가지 부가기능도 함께 제공한다.
- 해당 HTTP 요청이 시작부터 끝날 때 까지 유지되는 임시 저장소 기능
  - 저장: request.setAttribute(name, value)
  - 조회: request.getAttribute(name)
- 세션 관리 기능
  - request.getSession(create: true)

### HttpServletResponse 역할
- HTTP 응답 메시지 생성
  - HTTP 응답코드 지정
  - 헤더 생성
  - 바디 생성
 
 - 편의 기능 제공
   - Content-Type, 쿠키, Redirect

## 서블릿이 호출되는 과정


1. Servlet Request / Servlet Response 객체 생성
2. 설정 파일을 참고하여 매핑할 Servlet을 확인

```xml
<servlet>
    <servlet-name>MyServlet</servlet-name>
    <servlet-class>com.jsp.web.MyServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>MyServlet</servlet-name>
    <url-pattern>/hello</url-pattern>
</servlet-mapping>
```

3. 해당 서블릿 인스턴스 존재를 유무를 확인하여 없으면 생성(init())
4. Servlet Container에 스레드를 생성하고, res req를 인자로 service 실행


## 참고
https://youtu.be/calGCwG_B4Y

https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1
