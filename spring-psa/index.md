# Spring PSA


Spring의 대표적인 핵심가치 3가지로 IoC, AOP, PSA가 있다.
## PSA (Portable Service Abstraction)
 @Transactional 어노테이션을 선언하는 것 만으로 별도의 코드 추가 없이 트랜잭션 서비스를 사용할 수 있다. 내부적으로 트랜잭션 코드가 추상화되어 숨겨져 있는 것이다.
 이렇게 추상화 계층을 사용하여 어떤 기술을 내부에 숨기고 개발자에게 편의성을 제공해주는 것이 서비스 추상화(Service Abstraction)이며, **하나의 추상화로 여러 서비스를 묶어둔 것을 Spring에서 Portable Service Abstraction이라고 한다.**
 
 ## PSA의 원리
 트랜잭션을 예로 들어 설명하겠다.
 
 ![image](https://user-images.githubusercontent.com/46465928/154636355-8c87e181-91d4-4114-b18e-da6477466469.png)
위 그림처럼 Spring의 @Transactional은 각 TransactionManager를 각각 구현하고 있는 것이 아니라 최상위 PlatformTransactionManager를 이용하고 필요한 TransactionManager를 DI로 주입받아 사용하고 있다.

PlatformTransactionManager 인터페이스의 소스코드는 다음과 같다.
```java
public interface PlatformTransactionManager extends TransactionManager {

  TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;

  void commit(TransactionStatus status) throws TransactionException;

  void rollback(TransactionStatus status) throws TransactionException;
}
```

