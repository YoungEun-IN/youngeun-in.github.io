# 프록시 패턴 & 데코레이터 패턴


## 프록시 패턴

<img width="700" alt="스크린샷 2024-04-25 오후 9 36 19" src="https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/43c0f677-899e-4f0f-b1c3-4163edcaeb60">
<img width="700" alt="스크린샷 2024-04-25 오후 9 36 38" src="https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/01a278d2-15e6-4c20-b923-066cdc0d05c5">

```java
package hello.proxy.pureproxy.proxy.code;
public interface Subject {
    String operation();
}
```
예제에서 Subject 인터페이스는 단순히 operation() 메서드 하나만 가지고 있다.

```java
package hello.proxy.pureproxy.proxy.code;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class RealSubject implements Subject {
    @Override
    public String operation() {
        log.info("실제 객체 호출");
        sleep(1000);
        return "data";
    }
    private void sleep(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
RealSubject 는 Subject 인터페이스를 구현했다. operation() 은 데이터 조회를 시뮬레이션 하기 위해 1초 쉬도록 했다. 예를 들어서 데이터를 DB나 외부에서 조회하는데 1초가 걸린다고 생각하면 된다. 호출할 때 마다 시스템에 큰 부하를 주는 데이터 조회라고 가정하자.

```java
package hello.proxy.pureproxy.proxy.code;

import lombok.extern.slf4j.Slf4j;

public class ProxyPatternClient {
    private Subject subject;
    public ProxyPatternClient(Subject subject) {
        this.subject = subject;
    }
    public void execute() {
        subject.operation();
    }
}
```
Subject 인터페이스에 의존하고, Subject 를 호출하는 클라이언트 코드이다.
execute() 를 실행하면 subject.operation() 를 호출한다.

```java
package hello.proxy.pureproxy.proxy;

import lombok.extern.slf4j.Slf4j;
import hello.proxy.pureproxy.proxy.code.ProxyPatternClient;
import hello.proxy.pureproxy.proxy.code.RealSubject;
import hello.proxy.pureproxy.proxy.code.Subject;
import org.junit.jupiter.api.Test;

public class ProxyPatternTest {

    @Test
    void noProxyTest() {
        RealSubject realSubject = new RealSubject();
        ProxyPatternClient client = new ProxyPatternClient(realSubject);
        client.execute();
        client.execute();
        client.execute();
    }
}
```
테스트 코드에서는 client.execute() 를 3번 호출한다. 데이터를 조회하는데 1초가 소모되므로 총 3초의 시간이
걸린다.

```
RealSubject - 실제 객체 호출
RealSubject - 실제 객체 호출
RealSubject - 실제 객체 호출
```
결과는 위와 같다.

이미 개발된 로직을 전혀 수정하지 않고, 프록시 객체를 통해서 캐시를 적용해보자.

<img width="700" alt="스크린샷 2024-04-25 오후 9 42 04" src="https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/2a69bfa0-c296-470f-a92f-f62f623a150a">
<img width="700" alt="스크린샷 2024-04-25 오후 9 42 21" src="https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/f9179896-6994-4992-8443-b1dfcb7f4a16">

```java
package hello.proxy.pureproxy.proxy.code;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class CacheProxy implements Subject {
    private Subject target;
    private String cacheValue;
    public CacheProxy(Subject target) {
        this.target = target;
    }
    @Override
    public String operation() {
        log.info("프록시 호출");
        if (cacheValue == null) {
            cacheValue = target.operation();
        }
        return cacheValue;
    }
}
```
프록시도 실제 객체와 그 모양이 같아야 하기 때문에 Subject 인터페이스를 구현해야 한다.

```java
package hello.proxy.pureproxy.proxy;

import lombok.extern.slf4j.Slf4j;
import hello.proxy.pureproxy.proxy.code.CacheProxy;
import hello.proxy.pureproxy.proxy.code.ProxyPatternClient;
import hello.proxy.pureproxy.proxy.code.RealSubject;
import hello.proxy.pureproxy.proxy.code.Subject;
import org.junit.jupiter.api.Test;

public class ProxyPatternTest {
    @Test
    void noProxyTest() {
        RealSubject realSubject = new RealSubject();
        ProxyPatternClient client = new ProxyPatternClient(realSubject);
        client.execute();
        client.execute();
        client.execute();
    }
    @Test
    void cacheProxyTest() {
        Subject realSubject = new RealSubject();
        Subject cacheProxy = new CacheProxy(realSubject);
        ProxyPatternClient client = new ProxyPatternClient(cacheProxy);
        client.execute();
        client.execute();
        client.execute();
    }
}
```
realSubject 와 cacheProxy 를 생성하고 둘을 연결한다. 결과적으로 cacheProxy 가 realSubject 를 참조하는 런타임 객체 의존관계가 완성된다. 그리고 마지막으로 client 에 realSubject 가 아닌 cacheProxy 를 주입한다. 
이 과정을 통해서 client -> cacheProxy -> realSubject 런타임 객체 의존 관계가 완성된다.

cacheProxyTest() 는 client.execute() 을 총 3번 호출한다. 이번에는 클라이언트가 실제 realSubject를 호출하는 것이 아니라 cacheProxy 를 호출하게 된다.

```
CacheProxy - 프록시 호출
RealSubject - 실제 객체 호출
CacheProxy - 프록시 호출
CacheProxy - 프록시 호출
```
실행 결과는 위와 같다. 결과적으로 캐시 프록시를 도입하기 전에는 3초가 걸렸지만, 캐시 프록시 도입 이후에는 최초에 한번만 1초가 걸리고, 이후에는 거의 즉시 반환한다.

**프록시 패턴의 핵심은 클라이언트 코드의 변경 없이 자유롭게 프록시를 넣고 뺄 수 있다는 것이다. 실제 클라이언트 입장에서는 프록시 객체가 주입되었는지, 실제 객체가 주입되었는지 알지 못한다.**

## 프록시 패턴 vs 데코레이터 패턴

데코레이터 패턴의 의존관계는 다음과 같다.

<img width="700" alt="스크린샷 2024-04-25 오후 10 02 20" src="https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/81f146d1-ae87-43e5-9bdd-e128c56d03bd">
<img width="700" alt="스크린샷 2024-04-25 오후 10 02 37" src="https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/50614a7a-9d2c-4bab-8271-80c889817773">

이처럼 프록시 패턴과 데코레이터 패턴은 그 모양이 거의 같고, 상황에 따라 정말 똑같을 때도 있다. 

디자인 패턴에서 중요한 것은 해당 패턴의 겉모양이 아니라 그 패턴을 만든 의도이다. 따라서 의도에 따라 패턴을 구분한다.
- 프록시 패턴의 의도: 다른 개체에 대한 접근을 제어하기 위해 대리자를 제공
  - 권한에 따른 접근 차단
  - 캐싱
  - 지연 로딩
- 데코레이터 패턴의 의도: 객체에 추가 책임(기능)을 동적으로 추가하고, 기능 확장을 위한 유연한 대안 제공
  - 원래 서버가 제공하는 기능에 더해서 부가 기능을 수행
    - 예) 요청 값이나, 응답 값을 중간에 변형한다.
    - 예) 실행 시간을 측정해서 추가 로그를 남긴다.

즉 **프록시를 사용하고 해당 프록시가 접근 제어가 목적이라면 프록시 패턴이고, 새로운 기능을 추가하는 것이 목적이라면 데코레이터 패턴이 된다.**

## 참고
https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8

