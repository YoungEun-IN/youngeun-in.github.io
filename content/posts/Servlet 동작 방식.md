---
title: Servlet 동작 방식
date: 2022-02-23T15:12:26+09:00
categories:
  - java
tags: 
  - servlet-container
  - a-good-developer
---

## 서블릿이 호출되는 과정
`서블릿 컨테이너`는 서블릿을 담아 관리한다. 서블릿은 싱글톤으로 관리된다.

![image](https://user-images.githubusercontent.com/46465928/155554827-4e08d4c2-9239-4ec2-8522-c6c0b7ed0dac.png)

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
