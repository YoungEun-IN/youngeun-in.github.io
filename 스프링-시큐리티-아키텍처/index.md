# 스프링 시큐리티 주요 개념


다음은 스프링 시큐리티에서 사용하는 주요 개념에 대한 설명이다.

## SecurityContextHolder

**지정된 SecurityContext를 현재 실행 스레드와 연결한다.** 이 클래스는 SecurityContextHolderStrategy 인스턴스에 위임하는 일련의 정적 메서드를 제공한다. 

클래스의 목적은 주어진 JVM에 사용해야 하는 전략을 지정하는 편리한 방법을 제공하는 것이다. 기본적으로 ThreadLocal을 사용한다.

## SecurityContext
**현재 실행 스레드와 관련된 최소 보안 정보를 정의하는 인터페이스이다.** 보안 컨텍스트는 SecurityContextHolder에 저장되며, Authentication을 제공한다.

```java
public interface SecurityContext extends Serializable {
  Authentication getAuthentication();
  void setAuthentication(Authentication authentication);
}
```
## AuthenticationManager
**스프링 시큐리티에서 인증을 하는 주체이다.** 인자로 받은 Authentication이 유효한 인증인지 확인하고 Authentication 객체를 리턴한다.

사용자가 입력한 password가 UserDetailsService를 통해 읽어온 UserDetails 객체에 들어있는 password와 일치하는지, 해당 사용자 계정이 잠겨 있진 않은지, 비활성 계정은 아닌지 등 확인한다. 인증을 확인하는 과정에서 비활성 계정, 잘못된 비번, 잠긴 계정 등의 에러를 던질 수 있다.

```java
public interface AuthenticationManager {
  Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```
## Authentication
**AuthenticationManager.authenticate(Authentication) 메소드에 의해 요청이 처리된 후 인증 요청 또는 인증된 주체에 대한 토큰을 나타낸다.** 요청이 인증되면 인증은 일반적으로 사용 중인 인증 메커니즘에 의해 SecurityContextHolder가 관리하는 스레드 로컬 SecurityContext에 저장된다.

Principal과 Credentials, GrantAuthority 등을 제공한다.

```java
public interface Authentication extends Principal, Serializable {
	Collection<? extends GrantedAuthority> getAuthorities();
	Object getCredentials();
	Object getDetails();
	Object getPrincipal();
	boolean isAuthenticated();
	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```
### Principal
**개인, 회사 및 로그인 ID와 같은 모든 엔터티를 나타내는 데 사용할 수 있는 주체의 추상 개념을 나타낸다.** UserDetailsService에서 리턴한 객체이며 UserDetails 타입이다.

### GrantAuthority
**인증 개체에 부여된 권한을 나타낸다.** "ROLE_USER", "ROLE_ADMIN" 등으로 나타내며, 인증 이후, 인가 및 권한 확인할 때 이 정보를 참조한다.

## UserDetails
**애플리케이션이 가지고 있는 유저 정보와 스프링 시큐리티가 사용하는 Authentication 객체 사이의 어댑터이며, 핵심 사용자 정보를 제공한다.** 구현은 보안 목적으로 Spring Security에서 직접 사용되지 않는다. 나중에 인증 개체에 캡슐화되는 사용자 정보를 저장하기만 하면 된다. 이를 통해 보안과 관련되지 않은 사용자 정보(이메일 주소, 전화번호 등)를 편리한 위치에 저장할 수 있다.

```java
public interface UserDetails extends Serializable {
  Collection<? extends GrantedAuthority> getAuthorities();
  String getPassword();
  String getUsername();
  boolean isAccountNonExpired();
  boolean isAccountNonLocked();
  boolean isCredentialsNonExpired();
  boolean isEnabled();
}
```

## UserDetailsService
**유저 정보를 UserDetails 타입으로 가져오는 DAO(Data Access Object) 인터페이스이다.**

```java
public interface UserDetailsService {
  UserDetails loadUserByUsername(String username) throws UsernameNotFoundException; //사용자 이름을 기준으로 사용자를 찾는다.
}
```
세부 구현은 마음대로 가능하다. 실제 구현에서는 구현 인스턴스가 구성된 방식에 따라 검색이 대소문자를 구분하거나 대소문자를 구분하지 않을 수 있다.

## 참고
https://www.inflearn.com/course/%EB%B0%B1%EA%B8%B0%EC%84%A0-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0

