---
title: 테스트 더블(Test Double)
date: 2022-01-01T15:12:26+09:00
categories:
  - 개발방법론
tags: 
  - dummy
  - fake
  - stub
  - mock
---

테스트 더블(Test Double)이란 [xUnit Test Patterns](http://www.acornpub.co.kr/book/xunit)의 저자인 제라드 메스자로스(Gerard Meszaros)가 만든 용어로 테스트를 진행하기 어려운 경우 이를 대신해 테스트를 진행할 수 있도록 만들어주는 객체를 말한다. 테스트 더블이라는 용어는 영화 촬영 시 위험한 역할을 대신하는 스턴트 더블에서 비롯되었다.

예를 들어 우리가 데이터베이스로부터 조회한 값을 연산하는 로직을 구현했다고 하자. 해당 로직을 테스트하기 위해선 항상 데이터베이스의 영향을 받을 것이고, 이는 데이터베이스의 상태에 따라 다른 결과를 유발할 수도 있다.

이처럼 **테스트하려는 객체와 연관된 객체를 사용하기가 어렵고 모호할 때 대신해 줄 수 있는 객체를 테스트 더블이라 한다.**

![image](https://github.com/YoungEun-IN/youngeun-in.github.io/assets/46465928/2bbba983-7a2f-4d32-99f4-1bd70f0a3303)

## Dummy

- 인스턴스화 된 객체가 필요하지만 기능은 필요하지 않은 경우에 사용한다.
- Dummy 객체의 메서드가 호출되었을 때 정상 동작은 보장하지 않는다.
- 객체는 전달되지만 사용되지 않는 객체이다.

간단한 예시를 통해 알아보자.

```java
public interface PringWarning {
    void print();
}
```

```java
public class PrintWarningDummy implements PrintWarning {
    @Override
    public void print() {
        // 아무런 동작을 하지 않는다.
    }
}
```

실제 객체는 PrintWarning 인터페이스의 구현체를 필요하지만, 특정 테스트에서는 해당 구현체의 동작이 전혀 필요하지 않을 수 있다. 실제 객체가 로그용 경고만 출력한다면 테스트 환경에서는 전혀 필요 없기 때문이다.

이런 경우에는 `print()` 가 아무런 동작을 하지 않아도 테스트에는 영향을 미치지 않는다.

### Fake

- 복잡한 로직이나 객체 내부에서 필요로 하는 다른 외부 객체들의 동작을 단순화하여 구현한 객체이다.
- 동작의 구현을 가지고 있지만 실제 프로덕션에는 적합하지 않은 객체이다.

간단한 예시를 통해 알아보자.

```java
@Entity
public class User {
    @Id
    private Long id;
    private String name;
    
    protected User() {}
    
    public User(Long id, String name) {
        this.id = id;
        this.name = name;
    }
    
    public Long getId() {
        return this.id;
    }
    
    public String getName() {
        return this.name;
    }
}
```

```java
public interface UserRepository {
    void save(User user);
    User findById(long id);
}
```

```java
public class FakeUserRepository implements UserRepository {
    private Collection<User> users = new ArrayList<>();
    
    @Override
    public void save(User user) {
        if (findById(user.getId()) == null) {
            user.add(user);
        }
    }
    
    @Override
    public User findById(long id) {
        for (User user : users) {
            if (user.getId() == id) {
                return user;
            }
        }
        return null;
    }
}
```

테스트해야 하는 객체가 데이터베이스와 연관되어 있다고 가정해보자.

그럴 경우 실제 데이터베이스를 연결해서 테스트해야 하지만, 실제 데이터베이스 대신 가짜 데이터베이스 역할을 하는 FakeUserRepository를 만들어 테스트 객체에 주입하는 방법도 있다. 이렇게 하면 테스트 객체는 데이터베이스에 의존하지 않으면서도 동일하게 동작을 하는 가짜 데이터베이스를 가지게 된다.

### Stub

- 테스트에서 호출된 요청에 대해 미리 준비해둔 결과를 제공한다.

간단한 예시를 통해 살펴보자.

```java
public interface UserRepository {
    void save(User user);
    User findById(long id);
}
```

```java
public class StubUserRepository implements UserRepository {
    // ...
    @Override
    public User findById(long id) {
        return new User(id, "Test User");
    }
}
```

위의 코드처럼 StubUserRepository는 `findById()` 메서드를 사용하면 언제나 Test User라는 이름을 가진 User 인스턴스를 반환받는다.

테스트 환경에서 User 인스턴스의 name을 Test User만 받기를 원하는 경우 이처럼 동작하는 객체(UserRepository의 구현체)를 만들어 사용할 수 있다.

물론 이러한 방식의 단점은 테스트가 수정될 경우(`findById()` 메서드가 반환해야 할 값이 변경되야 할 경우) Stub 객체도 함께 수정해야 하는 단점이 있다.

```java
public interface UserRepository {
    void save(User user);
    User findById(long id);
}
```

SpringBoot에서의 Mockito 프레임워크는 다음과 같이 Stub을 검증할 수 있다.

```java
@ExtendWith(MockitoExtension.class)
public class UserServiceTest {
    @Mock
    private UserRepository userRepository;
    
    @Test
    void test() {
        when(userRepository.findById(anyLong())).thenReturn(new User(1, "Test User"));
        
        User actual = userService.findById(1);

        assertThat(actual.getId()).isEqualTo(1);
        assertThat(actual.getName()).isEqualTo("Test User");
    }
}
```

### Mock

- 객체의 행동을 사전에 설정하고, 그 행동이 제대로 호출되었는지를 검증한다.

SpringBoot에서의 Mockito 프레임워크는 다음과 같이 Mock을 검증할 수 있다.

```java
public interface UserRepository {
    void save(User user);
    User findById(long id);
}
```

```java
@ExtendWith(MockitoExtension.class)
public class UserServiceTest {
    @Mock
    private UserRepository userRepository;
    
    @Test
    void test() {
        User actual = userService.findById(1);

        verify(userRepository).findById(any());
    }
}
```

## 참고

https://tecoble.techcourse.co.kr/post/2020-09-19-what-is-test-double/
