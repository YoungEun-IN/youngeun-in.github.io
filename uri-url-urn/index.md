# URI, URL, URN


## URI

**URI는 로케이터(locator), 이름(name)을 포함한 개념**이다.

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/f328e043-4139-43a8-9749-c53f4e0c4629)

URI는 다음과 같이 풀이할 수 있다.
- Uniform: 리소스 식별하는 통일된 방식
- Resource: 자원, URI로 식별할 수 있는 모든 것(제한 없음)
- Identifier: 다른 항목과 구분하는데 필요한 정보

## URL, URN

URL, URN은 다음과 같이 풀이할 수 있다.
- URL: Uniform Resource Locator : **리소스가 있는 위치를 지정**
- URN: Uniform Resource Name : **리소스에 이름을 부여**
  - URN 이름만으로 실제 리소스를 찾을 수 있는 방법이 보편화 되지 않아, 잘 사용하지 않는다.
 
### URL 전체 문법

전체 문법은 다음과 같다.

**scheme://[userinfo@]host[:port][/path][?query][#fragment]**

- scheme : 주로 프로토콜 사용
  - 프로토콜: 어떤 방식으로 자원에 접근할 것인가 하는 약속 규칙
- userinfo : URL에 사용자정보를 포함해서 인증
  - 거의 사용하지 않음
- host : 호스트명, 도메인명 또는 IP 주소를 직접 사용가능
- port : 접속 포트
  - 일반적으로 생략, 생략시 http는 80, https는 443
- path : 리소스 경로(path), 계층적 구조
- query : key=value 형태
  - ?로 시작, &로 추가 가능
  - query parameter, query string 등으로 불림, 웹서버에 제공하는 파라미터, 문자 형태
- fragment : html 내부 북마크 등에 사용
  - 서버에 전송하는 정보 아님

## 참고

https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC

