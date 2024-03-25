# 트랜잭션 전파 설정


Spring에서 사용하는 어노테이션 '@Transactional'은 해당 메서드를 하나의 트랜잭션 안에서 진행할 수 있도록 만들어주는 역할을 한다. 이때 트랜잭션 내부에서 트랜잭션을 또 호출한다면 스프링에서는 새로운 트랜잭션이 생성될 수도 있고, 이미 트랜잭션이 있다면 부모 트랜잭션에 합류할 수도 있을 것이다. **진행되고 있는 트랜잭션에서 다른 트랜잭션이 호출될 때 어떻게 처리할지 정하는 것을 '트랜잭션의 전파 설정'이라고 부른다.**

## 전파 설정 옵션

트랜잭션의 전파 설정은 '@Transactional'의 옵션 'propagation'을 통해 설정할 수 있다. 각 옵션은 아래와 같다.

### REQUIRED (기본값)

부모 트랜잭션이 존재한다면 부모 트랜잭션으로 합류한다. 부모 트랜잭션이 없다면 새로운 트랜잭션을 생성한다. 중간에 롤백이 발생한다면 모두 하나의 트랜잭션이기 때문에 진행사항이 모두 롤백된다.

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/fa254eca-fe6f-4a8a-9665-a93007417fab)

### REQUIRES_NEW

무조건 새로운 트랜잭션을 생성한다. 각각의 트랜잭션이 롤백되더라도 서로 영향을 주지 않는다.

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/90efed4c-9cbf-477d-a5d2-2cb3e0123aec)

### MANDATORY

부모 트랜잭션에 합류한다. 만약 부모 트랜잭션이 없다면 예외를 발생시킨다.

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/040d6885-5672-4349-9628-23eb0aff465a)

### NESTED

부모 트랜잭션이 존재한다면 중첩 트랜잭션을 생성한다. 중첩된 트랜잭션 내부에서 롤백 발생시 해당 중첩 트랜잭션의 시작 지점 까지만 롤백된다. 중첩 트랜잭션은 부모 트랜잭션이 커밋될 때 같이 커밋된다. 부모 트랜잭션이 존재하지 않는다면 새로운 트랜잭션을 생성한다.

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/03c8201e-5fde-4e46-814c-056194faed8b)

### NEVER

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/de948309-75fd-40a2-adda-7bbccc822ae3)

### SUPPORTS

부모 트랜잭션이 있다면 합류한다. 진행중인 부모 트랜잭션이 없다면 트랜잭션을 생성하지 않는다.

### NOT_SUPPORTED

부모 트랜잭션이 있다면 보류시킨다. 진행중인 부모 트랜잭션이 없다면 트랜잭션을 생성하지 않는다.

## 참고

https://deveric.tistory.com/86

