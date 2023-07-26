# 영속성 컨텍스트


## 영속성 컨텍스트

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/73759d75-ac84-48a8-aa0a-1aff79049243)

처음 생성된 **엔티티 매니저 팩토리는 클라이언트의 요청에 따라 엔티티 매니저를 생성**한다. 그리고 **엔티티 매니저는 DB Connection Pool에 있는 Connection을 이용해서 DB에 접근**한다.

**영속성 컨텍스트란 엔티티를 영구 저장하는 환경**이다. 영속성 컨텍스트는 논리적인 개념이며, 눈에 보이지 않는다. 엔티티 매니저를 통해서 영속성 컨텍스트에 접근한다.

## 엔티티의 생명주기

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/bde4bba3-3140-4ae0-89bd-c62c880cf5bc)

- **비영속(new/transient)**
  - 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태.
  - 객체를 단순히 생성한 상태
- **영속(managed)**
  - 영속성 컨텍스트에 의해 관리되는 상태
  - 객체를 생성한 뒤 엔티티 매니저를 통해 persist 해주면 영속 상태가 된다.
- **준영속(detached)**
  - 영속성 컨텍스트에 저장되었다가 분리된 상태
  - `em.detach(member);`
- **삭제(removed)**
  - 삭제된 상태
  - `em.remove(member);`
 
## 영속성 컨텍스트의 이점
### 1차 캐시
1차 캐시를 통해 DB에서의 접근을 줄일 수 있게 된다.

영속된 엔티티들은 영속 컨텍스트의 1차캐시에 저장되기 때문에 이미 1차캐시에 있는 엔티티를 찾기 위해서 쿼리를 실행할 필요가 없게 된다.

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/1acd6deb-d4dd-4141-902e-ff7c586bb933)

persist를 통해 엔티티를 영속 컨텍스트에 저장하면 1차 캐시에 저장되며, find를 통해 조회할 때 캐시를 통해 조회하게 된다. 

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/2924f9cd-77b3-4b72-aab1-d07429e596ad)

member2를 조회하게 되면 1차 캐시에 없기 때문에 DB에서 조회(SELECT문 실행)를 해서 1차 캐시에 저장하게 됩니다. 이후 member2를 반환한다.

### 영속 엔티티의 동일성 보장
1차 캐시를 통해 REPEATABLE READ를 가능하게 만들어준다.

### 트랜잭션을 지원하는 쓰기 지연(Transactional write-behind)
쓰기 지연 SQL 저장소를 통해 버퍼링을 가능하게 한다.

트랜잭션 내에서의 persist를 호출할 때마다 INSERT SQL을 데이터베이스에 보내는 게 아니라 커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/90b02e86-6576-49c5-95ed-35ce0f5b5d8f)

영속 컨텍스트에는 쓰기 지연 SQL 저장소라는 것이 존재한다. em.persist(memberA)를 호출 시 1차 캐시에 memberA를 저장함과 동시에 INSERT memberA SQL을 쓰기 지연 SQL 저장소에 동시에 저장한다. 이후 memberB도 같은 방식으로 1차 캐시와 동시에 쓰기 지연 SQL 저장소에 SQL을 저장한다.

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/ff0c833d-2f92-4796-9b65-492813bb9801)

쓰기 지연 SQL 저장소에 쌓인 SQL은 DB의 commit이나 flush함수 호출 시 실행된다. SQL이 실행된 이후 DB에 값이 저장된다. 

### 변경 감지(Dirty Checking)
영속성 컨텍스트를 통해 변경을 감지할수도 있다.

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/e7324676-85f0-41d9-9b7c-fd4d72caf9b4)

영속 상태의 엔티티는 값을 변경한 후 update와 같은 작업이 없어도, flush나 commit 시점에 엔티티와 맨 처음 영속 컨텍스트에 들어올 때 생긴 스냅샷의 비교를 통해 영속 상태의 변경이 있을 시 엔티티 값을 변경하는 UPDATE 쿼리를 DB에 날린다.

### 지연 로딩(Lazy Loading)
연관된 엔티티를 실제 사용하는 시점에 데이터베이스에서 조회할 수 있다.

## 참고
https://www.inflearn.com/course/ORM-JPA-Basic

