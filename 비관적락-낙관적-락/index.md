# 비관적락, 낙관적 락


## 비관적 락
비관적 락은 **대부분의 경우 트랜젝션에서 충돌이 발생할 것이라 가정하고, 일단 락을 걸고 보는 기법**이다. 락을 걸게 되면 다른 트랜잭션에서는 락이 해제되기 전에 데이터를 가져갈 수 없다. 비관적 락은 데이터베이스의 락 알고리즘을 통해 구현되며, 데이터를 수정하는 즉시 트랜잭션 충돌을 감지한다.

### JPA에서 비관적 락 구현
```java
public interface HomeRepository extends JpaRepository<Home, Long> {

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("select h from Home h where h.name = :name")
    Home findWithNameForUpdate(@Param("name") String name);
}
```

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class HomeService {
  private final HomeRepository homeRepository;

  @Transactional
  public int decreasePrice(String name, int price) {
      Home home = homeRepository.findWithNameForUpdate(name);
      home.decreasePrice(price);
      return home.getPrice();
  }
}
```

```sql
Hibernate:
    select
        home0_.idx as idx1_0_,
        home0_.address as address2_0_,
        home0_.name as name3_0_,
        home0_.price as price4_0_
    from
        home home0_
    where
        home0_.name=? for update
```

### 비관적 락의 LockModeType
- PESSIMISTIC_READ
  - shared lock을 얻을 수 있다. 데이터를 반복 읽기만 하고 수정하지 않는 용도로 락을 걸 때 사용하며, 일반적으로 잘 사용하지 않는다. 데이터베이스 대부분은 방언에 의해 PESSIMISTIC_WRITE로 동작한다.
- PESSIMISTIC_WRITE
  - exclusive lock을 얻을 수 있다. 데이터베이스에 쓰기 락을 걸 때 사용한다.

## 낙관적 락
낙관적 락은 **대부분의 경우 트랜젝션의 충돌이 일어나지 않는다는 낙관적 가정을 하는 기법**이다. 낙관적 락은 실제로 Lock을 이용하지 않고 버전을 이용함으로써 정합성을 맞추는 방법이다. 

낙관적 락은 데이터베이스 자체에서 제공하는 것이 아니라 어플리케이션에서 제공하는 기능이다. 먼저 데이터를 읽은 후에 update를 수행할 때 현재 내가 읽은 버전이 맞는지 확인하여 업데이트한다. 내가 읽은 버전에서 수정사항이 생겼을 때에는 application에서 다시 읽은 후에 작업을 수행한다.

### JPA에서 낙관적 락 구현
JPA에서 낙관적 락을 사용하기 위해서는 엔티티 클래스에 @Version 어노테이션을 사용해서 버전 관리 기능을 추가해야 한다.

```java
@Entity
public class Home {

 @Id 
 private Long id;
 private String name;
 private int price;

 @Version
 private Integer version;
}
```

```java
public interface HomeRepository extends JpaRepository<Home, Long> {

    @Lock(LockModeType.OPTIMISTIC)
    @Query("select h from Home h where h.name = :name")
    Home findWithNameForUpdate(@Param("name") String name);
}
```

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class HomeService {
  private final HomeRepository homeRepository;

  @Transactional
  public int decreasePrice(String name, int price) {
      Home home = homeRepository.findWithNameForUpdate(name);
      home.decreasePrice(price);
      return home.getPrice();
  }
}
```

@Version을 명시할때는 다음과 같은 주의사항이 있다.
 - 각 엔티티 클래스에는 하나의 버전 속성 만 있어야 한다.
 - 여러 테이블에 매핑 된 엔티티의 경우 기본 테이블에 배치되어야 한다.
 - 버전에 명시할 타입은 int, Integer, long, Long, short, Short, java.sql.Timestamp 중 하나여야 한다.

JPA는 Select시에 트랜잭션 내부에 버전 속성의 값을 보유하고 트랜젝션이 업데이트를 하기 전에 버전 속성을 다시 확인한다.
그 동안에 버전 정보가 변경이 되면 `OptimisticLockException`이 발생하고 변경되지 않으면 트랜잭션은 버전속성을 증가하는 업데이트 하게 된다.

### 낙관적 락의 LockModeType
- NONE
  - **별도의 옵션을 사용하지 않아도 Entity에 @Version이 적용된 필드만 있으면 낙관적 락이 적용**된다. Entity 수정시에 낙관적 락이 발생한다.
- OPTIMISTIC
  - **Entity 수정시에만 발생하는 낙관적 락이 읽기 시에도 발생하도록 설정**한다. 
  - 읽기시에도 버전을 체크하고 트랜잭션이 종료될 때까지 다른 트랜잭션에서 변경하지 않음을 보장한다. 이를 통해 dirty read와 non-repeatable read를 방지한다.
- OPTIMISTIC_FORCE_INCREMENT
  - **엔티티를 수정하지 않아도 트랜잭션을 커밋할 때 UPDATE 쿼리를 사용해서 버전 정보를 강제로 증가**시킨다. 이때 데이터베이스의 버전이 엔티티의 버전과 다르면 예외가 발생한다. 추가로 엔티티를 수정하면 수정시 버전 UPDATE 가 발생하여 총 2번의 버전 증가가 나타날 수 있다.
  - 논리적인 단위의 엔티티 묶음을 관리할 수 있다. 예를 들어 게시물과 첨부파일이 일대다, 다대일의 양방향 연관관계이고 첨부파일이 연관관계의 주인일 때, 첨부파일 추가 시 게시물의 버전도 강제로 증가해야 할 때 사용할 수 있다.
 
## 비관적 락 vs 낙관적 락
||비관적 락|낙관적 락|
|------|---|---|
|장점|- 업데이트가 실패했을 때 재시도 로직을 작성하지 않아도 된다.|- 실제 Lock을 이용하지 않으므로 비관적 락보다 성능상 이점이 있다.|
|단점|- 낙관적 락보다 성능이 좋지 않다.<br>- 데드락이 걸릴 수 있다.|- 업데이트가 실패했을 때 재시도 로직을 개발자가 직접 작성해야 한다.|

결론적으로, 충돌이 빈번하게 일어난다면 비관적 락, 빈번하지 않다면 낙관적 락을 사용하는 것이 좋다.

## 참고
https://www.inflearn.com/course/%EB%8F%99%EC%8B%9C%EC%84%B1%EC%9D%B4%EC%8A%88-%EC%9E%AC%EA%B3%A0%EC%8B%9C%EC%8A%A4%ED%85%9C

https://devjem.tistory.com/28

https://sup2is.github.io/2020/10/22/jpa-optimistic-lock-and-pessimistic-lock.html

https://effectivesquid.tistory.com/entry/Optimistic-Lock%EA%B3%BC-Pessimistic-Lock

