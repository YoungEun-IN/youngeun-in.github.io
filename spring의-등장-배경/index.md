# Spring의 등장 배경


## EJB(Enterprise Java Beans)와 POJO(Plain Old Java Object)
### EJB(Enterprise Java Beans)
#### 개념

- 기업환경의 시스템을 구현하기 위한 서버측 컴포넌트 모델
- 애플리케이션의 업무 로직을 가지고 있는 서버 애플리케이션
- Java EE의 자바 API 중 하나

#### 특징

- 동시 접속자 수가 많은 가운데 안정적인 트랜잭션이 필요한 사이트 구축시 사용하는 컴포넌트 기술
  - ex) 공공기관, 기상청, 병무청, 금융, 보험, 포털사이트, 게임사이트, 기업 등..
- JSP, Beans를 사용한 시스템보다 속도는 느리지만, **개발시 개발자에게 많은 자동화된 기능을 제공하여 안정적인 분산 시스템 구축 및 제공**을 용이하게 함
- 기초 기술(JSP, BEANS, RMI, Servlet, Serialization, Transaction, Connection Pooling)을 알면 학습 및 사용이 쉬움
- EJB 규약을 집중적으로 습득하면 쉽게 EJB 컴포넌트 개발 가능

#### 장단점
|장점|단점|
|---|---|
|많은 동시접속자에 대한 안정성 지원|객체지향적이지 않음|
|처리 메소드를 컨테이너가 트랜잭션 자동 처리|복잡한 프로그래밍 모델|
|빈즈의 상태를 메모리에서 사용 여부에 따라 자동으로 활성화/비활성화 실행하여 관리|특정 환경·기술에 종속적인 코드|
|다층 구조 시스템 구축 가능|컨테이너 안에서만 동작할 수 있는 객체 구조|
|라이프 사이클, 보안, Threading 등의 서비스 제공|부족한 개발생산성 및 이동성|

### POJO(Plain Old Java Object)
- 특정 자바 모델이나 기능, 프레임워크 등에 종속되지 않고, 클래스 패스(class path)를 필요로 하지 않는 일반적인 Java Object
- 중량 프레임워크들(Java EE 등...)을 사용하게 되면서 해당 프레임워크에 종속된 **"무거운" 객체를 만들게 된 것에 반발**해서 사용하게 됨
- 2000년 9월 마틴 파울러, 레베카 파슨, 조쉬 맥킨지 등에 의해 사용되기 시작함

## Spring과 Hibernate의 등장
EJB의 단점들로 인해 대표적으로 2가지의 Open Source가 등장하게 되었다.

### Spring
- 2002년 출간된 로드 존슨의 『expert one-on-one J2EE Design and Development』이 Spring의 근간이 됨
- **EJB 컨테이너를 대체**하게 됨
- 사실상 현재의 표준 기술

### Hibernate
- 자바 언어를 위한 객체 관계 매핑 프레임워크
- 객체 지향 도메인 모델을 관계형 데이터베이스로 매핑하기 위한 프레임워크 제공
- 2001년 Gavin King과 시러스 테크놀로지스 출신 동료들이 EJB2 스타일의 엔티티 빈즈 이용을 대체할 목적으로 출발
- 기술력이 낮았던 **EJB Entitiy Bean 기술을 대체**
- JPA(Java Persistence API)의 새로운 표준 정의

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/02bce1b5-a4c6-4cbf-9039-fe3d36502938)

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/1df085b8-8ca2-478d-998f-c7d6172086e3)

## 스프링의 역사
|년도|버전|설명|
|------|---|---|
|2003년|스프링 프레임워크 1.0|XML 기반|
|2006년|스프링 프레임워크 2.0|XML 편의 기능 지원|
|2009년|스프링 프레임워크 3.0|자바 코드로 설정|
|2013년|스프링 프레임워크 4.0|자바8|
|2014년|스프링 부트 1.0|스프링 프레임워크의 설정 및 배포에 대한 어려움 해결하기 위해 등장|
|2017년|스프링 프레임워크 5.0, 스프링 부트 2.0|리액티브 프로그래밍(비동기) 지원|
|2022년|스프링 프레임워크 6.0, 스프링 부트 3.0|Java17과 Java19 지원, GraalVM 지원|

## 참고
https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8

https://velog.io/@yu-jin-song/Spring-%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5-%EC%84%A4%EA%B3%84%EC%99%80-%EC%8A%A4%ED%94%84%EB%A7%81



