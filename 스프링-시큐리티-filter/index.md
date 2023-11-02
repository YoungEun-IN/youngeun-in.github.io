# 스프링 시큐리티 Filter


## DelegatingFilterProxy와 FilterChainProxy

### DelegatingFilterProxy
일반적인 서블릿 필터이며, **서블릿 필터 처리를 스프링에 들어있는 빈으로 위임할 때 사용한다.** 타겟 빈 이름을 설정할 수 있다.

스프링 부트 없이 스프링 시큐리티 설정할 때는 AbstractSecurityWebApplicationInitializer를 사용해서 등록해서 사용하며, 스프링 부트를 사용할 때는 자동으로 등록 된다. (SecurityFilterAutoConfiguration)

### FilterChainProxy
**스프링부트에서 "springSecurityFilterChain" 이라는 이름의 빈으로 등록된다.**

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/cb51c561-22ca-4081-86e6-1bbcea636628)

## 스프링 시큐리티 Filter
스프링 시큐리티가 제공하는 필터들은 다음과 같다. 이 모든 필터는 FilterChainProxy가 호출한다.

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/f0246da8-0134-4709-aa26-d16864af7996)

### WebAsyncManagerIntegrationFilter
**스프링 MVC의 Async 기능을 사용할 때에도 SecurityContext를 공유하도록 도와주는 필터이다.**

@Async를 사용한 서비스를 호출하는 경우에는 쓰레드가 다르기 때문에 SecurityContext를 공유받지 못한다. (기본은 MODE_THREADLOCAL 전략)

```java
SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_INHERITABLETHREADLOCAL);
```

위와 같이 SecurityContextHolder 전략을 변경하면 @Async를 처리하는 쓰레드에서도 SecurityContext를 공유받을 수 있다.

### SecurityContextPersistenceFilter
**SecurityContextRepository를 사용해서 기존의 SecurityContext를 읽어오거나 초기화 하는 필터이다.** 기본으로 사용하는 전략은 HTTP Session을 사용한다.

### HeaderWriterFilter
**응답 헤더에 시큐리티 관련 헤더를 추가해주는 필터이다.**
- XContentTypeOptionsHeaderWriter: 마임 타입 스니핑 방어
  - X-Content-Type-Options: nosniff
- XXssProtectionHeaderWriter: 브라우저에 내장된 XSS 필터 적용
  - X-XSS-Protection: 1; mode=block
- CacheControlHeadersWriter: 캐시 히스토리 취약점 방어
  - Cache-Control: no-cache, no-store, max-age=0, must-revalidate
  - Expires: 0
  - Pragma: no-cache
- HstsHeaderWriter: HTTPS로만 소통하도록 강제
  - Strict-Transport-Security: max-age=31536000 ; includeSubdomains ; preload
    - max-age : 브라우저가 HSTS 정책을 적용할 기간(초)을 설정한다.
    - includeSubdomains : 도메인(example.com)의 서브 도메인(api.example.com 등)에도 HSTS 설정을 적용한다.
    - preload : 해당 도메인이 브라우저의 preload list에 추가되며, 브라우저에서는 HSTS 헤더가 없더라도 HTTP요청을 HTTPS로 강제 변환하여 전송한다.
- XFrameOptionsHeaderWriter: clickjacking 방어
  - X-Frame-Options: DENY

### CsrfFilter
이 기능은 인증과는 별도로 **악의적인 공격자의 침입을 막는 필터이다.** 클라이언트가 POST, PUT, DELETE 등의 HTTP METHOD 방식으로 요청할 때 반드시 csrf 토큰을 요구하게 되고 토큰이 없을 경우 서버 접근을 막는다.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.csrf().disable();
  }
}
```
설정 클래스에서 CSRF 기능을 비활성화 하면 CsrfFilter 필터 클래스가 동작하지 않으므로 세션이 생성되지 않는다.

### LogoutFilter
**로그아웃을 처리하는 필터이다.** LogoutHandler를 사용하여 로그아웃시 필요한 처리를 하며 이후에는 LogoutSuccessHandler를 사용하여 로그아웃 후처리를 한다.
- LogoutHandler
  - CsrfLogoutHandler : csrfTokenRepository 에서 csrf 토큰을 삭제한다.
  - SecurityContextLogoutHandler : 세션과 SecurityContext 를 삭제한다.
- LogoutSuccessHandler
  - SimplUrlLogoutSuccessHandler : 로그아웃 페이지로 리다이렉트한다.

```java
http.logout()
  .logoutUrl("/logout")
  .logoutSuccessUrl("/")
  .logoutRequestMatcher()
  .invalidateHttpSession(true)
  .deleteCookies()
  .addLogoutHandler()
  .logoutSuccessHandler();
```

### UsernamePasswordAuthenticationFilter
**폼 로그인을 처리하는 인증 필터이다.** 사용자가 폼에 입력한 username과 password로 Authentcation을 만들고
AuthenticationManager를 사용하여 인증을 시도한다.

AuthenticationManager (ProviderManager)는 여러 AuthenticationProvider를 사용하여
인증을 시도하는데, 그 중에 DaoAuthenticationProvider는 UserDetailsServivce를
사용하여 UserDetails 정보를 가져와 사용자가 입력한 password와 비교한다.

### DefaultLoginPageGeneratingFilter
**기본 로그인 폼 페이지를 생성해주는 필터이다.** GET /login 요청을 처리한다.

```java
http.formLogin()
  .usernameParameter("my-username")
  .passwordParameter("my-password")
  .loginPage("/signin");
```

### DefaultLogoutPageGeneratingFilter
**기본 로그아웃 페이지를 생성해주는 필터이다.**

### BasicAuthenticationFilter
**Basic 인증을 처리하는 필터이다.**

{{< admonition note "Basic 인증" true >}}
요청 헤더에 username와 password를 Base64 인코딩하여 실어 보내면 브라우저 또는 서버가 그 값을 읽어서 인증하는 방식이다.
예) Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l

브라우저 기반 요청이 클라이언트의 요청을 처리할 때 자주 사용하며, 보안에 취약하기 때문에 반드시 HTTPS를 사용할 것을 권장한다.
{{< /admonition >}}

### RequestCacheAwareFtiler
**현재 요청과 관련 있는 캐시된 요청이 있는지 찾아서 적용하는 필터이다.** 캐시된 요청이 없다면 현재 요청을 처리하고 캐시된 요청이 있다면 해당 캐시된 요청을 처리한다.

### SecurityContextHolderAwareReqeustFilter
**시큐리티 관련 서블릿 API를 구현해주는 필터이다.**
- HttpServletRequest#authenticate(HttpServletResponse)
- HttpServletRequest#login(String, String)
- HttpServletRequest#logout()
- AsyncContext#start(Runnable)

### AnonymouseAuthenticationFilter
**익명 사용자를 인증하는 필터이다.** SecurityContext에 저장된 Authentication이 null이면 익명 Authentication을 만들어 넣어주고, null이 아니면 아무일도 하지 않는다.

기본으로 만들어 사용할 익명 Authentication 객체를 설정할 수 있다.

```java
http.anonymous()
  .principal()
  .authorities()
  .key()
```

### SessionManagementFilter
**세션을 관리하는 필터이다.**

- 세션 변조 방지 전략 설정: sessionFixation
  - none
  - newSession
  - migrateSession (서블릿 3.0- 컨테이너 사용시 기본값)
  - changeSessionId (서브릿 3.1+ 컨테이너 사용시 기본값)
- 유효하지 않은 세션을 리다이렉트 시킬 URL 설정
  - invalidSessionUrl
- 동시성 제어: maximumSessions
  - 추가 로그인을 막을지 여부 설정 (기본값, false)
- 세션 생성 전략: sessionCreationPolicy
  - IF_REQUIRED : 스프링 시큐리티가 필요 시 생성(default)
  - NEVER : 스프링 시큐리티가 생성하지 않지만 이미 존재하면 사용
  - STATELESS : 스프링 시큐리티가 생성하지 않고 존재해도 사용하지 않음
  - ALWAYS : 스프링 시큐리티가 항상 세션 생성

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.정책상수);
  }
}
```

### RememberMeAuthenticationFilter
**세션이 사라지거나 만료가 되더라도 쿠키 또는 DB를 사용하여 저장된 토큰 기반으로 인증을 지원하는 필터이다.**

```java
http.rememberMe()
  .userDetailsService(accountService)
  .key("remember-me-sample");
```

### ExeptionTranslationFilter
**필터 체인에서 발생하는 AuthenticationException과 AccessDeniedException을 처리하는 필터이다.**

#### AuthenticationException
AuthenticationManager의 authenticate 메소드 실행 시 발생하는 예외이다. 비활성 계정, 잘못된 비밀번호, 잠긴 계정일 때 발생한다.

```java
  public interface AuthenticationManager {
  Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```

AuthenticationEntryPoint를 실행하며, AuthenticationEntryPoint는 인증 예외가 발생했을시 어떻게 할 것인지를 결정한다.

#### AccessDeniedException
익명 사용자라면 AuthenticationEntryPoint를 실행하며, 익명 사용자가 아니면 예외를 AccessDeniedHandler에게 위임한다.

```java
http.exceptionHandling()
  .accessDeniedHandler((request, response, accessDeniedException) -> {
    UserDetails principal = (UserDetails)
    SecurityContextHolder.getContext().getAuthentication().getPrincipal();
    String username = principal.getUsername();
    String servletPath = request.getServletPath();
    System.out.println(username + " is denied to access to " + servletPath);
    response.sendRedirect("/access-denied");
});
```

### FilterSecurityInterceptor
**AccessDecisionManager를 사용하여 Access Control 또는 예외 처리 하는 필터이다.** 대부분의 경우 FilterChainProxy에 제일 마지막 필터로 들어있다.

```java
http.authorizeRequests()
  .mvcMatchers("/", "/info", "/account/**", "/signup").permitAll()
  .mvcMatchers("/admin").hasAuthority("ROLE_ADMIN")
  .mvcMatchers("/user").hasRole("USER")
  .anyRequest().authenticated()
  .expressionHandler(expressionHandler());
```

{{< admonition note "AccessDecisionManager" true >}}
AccessDecisionManager는 Access Control 결정을 내리는 인터페이스로, 구현체 3가지를 기본으로 제공한다.
  - AffirmativeBased: 여러 Voter중에 한명이라도 허용하면 허용. 기본 전략.
  - ConsensusBased: 다수결
  - UnanimousBased: 만장일치
{{< /admonition >}}

## 참고
https://www.inflearn.com/course/%EB%B0%B1%EA%B8%B0%EC%84%A0-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0 

