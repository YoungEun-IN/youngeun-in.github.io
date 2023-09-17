# 스프링 시큐리티 필터 처리 동작방식


## CsrfFilter

이 기능은 인증과는 별도로 악의적인 공격자의 침입을 막기 위한 목적으로 **클라이언트가  POST, PUT, DELETE 등의 HTTP METHOD 방식으로 요청할 때 반드시 csrf 토큰을 요구하게 되고 토큰이 없을 경우 서버 접근을 막는다.**

모든 요청에 랜덤하게 생성된 토큰을 HTTP 파라미터로 요구하며,  요청 시 전달되는 토큰 값과 서버에 저장된 실제 값과 비교한 후 만약 일치하지 않으면 요청은 실패한다. 

스프링 시큐리티는 만약 클라이언트에서 요청시 전송한 CSRF 토큰이 존재할 경우 세션을 생성하고 세션에 토큰을 저장한다. 만약 클라이언트가 전송한 csrf 토큰이 없을 경우, 그리고 이미 존재하는 세션이 없을 경우에는 세션을 생성하지 않고 토큰을 저장하지 않는다.

```java
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf().disable();
    }
```

**설정 클래스에서 CSRF 기능을 비활성화 하면 CsrfFilter 필터 클래스가 동작하지 않으므로 세션이 생성되지 않는다.**

## SessionCreationPolicy

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.정책상수);
    }
```

스프링 시큐리티 세션 생성 정책은 다음과 같다.

- SessionCreationPolicy.Always : 스프링 시큐리티가 항상 세션 생성
- SessionCreationPolicy.IF_REQUIRED : 스프링 시큐리티가 필요 시 생성(default)
- SessionCreationPolicy.Never : 스프링 시큐리티가 생성하지 않지만 이미 존재하면 사용
- SessionCreationPolicy.Stateless: 스프링 시큐리티가 생성하지 않고 존재해도 사용하지 않음

**SessionCreationPolicy.STATELESS 는 스프링 시큐리티가 인증 메커니즘을 진행하는 과정에서 세션을 생성하지 않는다는 것이고 혹 이미 이전에 세션이 존재한다고 하더라도 그 세션을 사용하여 세션쿠기 방식의 인증처리를 하지 않겠다는 의미이다.**

`SessionCreationPolicy.STATELESS` 상태에서의 내부 처리 프로세스는 다음과 같다.

1. 클라이언트가 form 인증 방식으로 최초로 인증을 시도하게 되면 `UsernamePasswordAuthenticationFilter`가 인증 처리를 하게 되고 인증이 성공하면 `SecurityContext` 에 `Authentication` 객체를 저장하고 인증처리는 완료된다.
Authentication 객체에는 인증성공결과(User, Authorities)가 저장되어 있다.

2. 인증에 성공한 이후에 `SecurityContext` 객체를 세션에 저장하는 역할을 하는 `SecurityContextPersistenceFilter`는 `SessionCreationPolicy.STATELESS` 설정으로 인해 세션 존재 여부를 떠나서 세션을 통한 인증 메커니즘 방식을 취하지 않는다.

3. 인증에 성공한 이후라도 클라이언트가 다시 어떤 자원에 접근을 시도할 경우 `SecurityContextPersistenceFilter`는 세션 존재 여부를 무시하고 항상 새로운 `SecurityContext` 객체를 생성한다. 그렇기 때문에 매번 인증을 받아야 하고 인증을 필요로 하는 상태가 된다.

## 참고

https://www.inflearn.com/questions/34886/jsessionid%EC%9D%98-%EC%83%9D%EC%84%B1%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C

