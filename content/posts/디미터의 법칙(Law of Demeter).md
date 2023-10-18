---
title: 디미터의 법칙(Law of Demeter)
date: 2022-03-08T15:11:26+09:00
categories:
  - 설계방법론
tags: 
  - oop
  - law-of-demeter
---

## 디미터의 법칙(Law of Demeter)
디미터의 법칙은 “Object-Oriented Programming: An Objective Sense of Style” 에서 처음으로 소개되었다. Demeter라는 프로젝트를 진행하던 개발자들은 어떤 객체가 다른 객체에 대해 지나치게 많이 알다보니, 결합도가 높아지고 좋지 못한 설계를 야기한다는 것을 발견하였다. 그래서 이를 개선하고자 객체에게 **자료를 숨기는 대신 함수를 공개**하도록 하였는데, 이것이 바로 디미터의 법칙이다.

즉, 디미터의 법칙은 **다른 객체가 어떠한 자료를 갖고 있는지 속사정을 몰라야 한다**는 것을 의미하며, 이러한 이유로 Don’t Talk to Strangers(낯선 이에게 말하지 마라) 또는 Principle of least knowledge(최소 지식 원칙) 으로도 알려져 있다.

또는 직관적으로 이해하기 위해 **여러 개의 .(도트)을 사용하지 말라는 법칙**으로도 많이 알려져 있으며, 디미터의 법칙을 준수함으로써 캡슐화를 높혀 객체의 자율성과 응집도를 높일 수 있다.

객체 지향 프로그래밍에서 가장 중요한 것은 "객체가 어떤 데이터를 가지고 있는가?"가 아니라, **"객체가 어떤 메세지를 주고 받는가?"** 이다. 그렇기에 디미터의 법칙은 객체 지향 프로그래밍에서 상당히 중요한 개념인데, 디미터의 법칙이 위배된다는 것은 올바른 객체 지향 프로그래밍을 하지 못하고 있다는 증거이기도 하다.

## 디미터(Demeter)의 법칙 예시
### 디미터의 법칙을 위반하는 코드
예를 들어 서울에 살고 있는 어떤 사용자에게 알림을 보내주는 함수를 구현한다고 하자. 이를 구현하기 위해 우리는 다음과 같은 User 객체와 Address 객체를 필요로 할 것이고, User객체는 Address라는 주소 객체를 가지고 있을 것이다.

```java
@Getter
public class User {
    private String email;
    private String name;
    private Address address;
}

@Getter
public class Address {
    private String region;
    private String details;
}
```

이제 어떤 사용자가 서울에 살고 있으면 알림을 보내주는 함수를 다음과 같이 구현하였다고 하자.

```java
@Service
public class NotificationService {
    public void sendMessageForSeoulUser(final User user) {
        if ("서울".equals(user.getAddress().getRegion())) {
            sendNotification(user);
        }
    }
}
```

위와 같이 구현된 코드는 정말 흔하게 볼 수 있는 코드이지만, 디미터의 법칙을 위반하고 있는 코드이며 객체지향스럽지 못하다. 왜냐하면 우리는 객체에게 메세지를 보내는 것이 아니라 객체가 가지는 자료를 확인하고 있으며, 다른 객체가 어떠한 자료를 갖고 있는지 지나치게 잘 알기 때문이다. 우리는 Getter 메소드를 통해 user객체가 email, name, address를 가지고 있음을 파악할 수 있다. 그렇기에 우리는 위의 코드를 객체 지향스러우며 디미터의 법칙을 준수하도록 수정할 필요가 있다.

### 디미터의 법칙을 준수하는 코드

우리는 Address 객체의 데이터를 통해 사용자의 지역을 파악하는 것이 아니라, Address의 객체에 메세지를 보내서 서울 지역에 사는지 파악하도록 구현해야 한다.

```java
public class Address {
    private String region;
    private String details;

    public boolean isSeoulRegion() {
        return "서울".equals(region);
    }
}

public class User {
    private String email;
    private String name;
    private Address address;

    public boolean isSeoulUser() {
        return address.isSeoulRegion();
    }
}
```

위와 같이 객체에게 보내는 메세지를 구현하면 불필요한 @Getter들 역시 지워줄 수 있고, User 객체와 Address 객체가 어떠한 데이터들을 지니고 있는지 모른 채 메세지를 보낼 수 있다.

그리고 기존의 알림을 보내는 로직을 다음과 같이 수정할 수 있다. 

```java
@Service
public class NotificationService {
    public void sendMessageForSeoulUser(final User user) {
        if (user.isSeoulUser()) {
            sendNotification(user);
        }
    }
}
```

그리고 새롭게 작성된 코드를 살펴보면, 기존의 user.getAddress().getRegion() 처럼 여러 개의 .(도트)을 사용하여 참조하지 않기 때문에 디미터의 법칙을 잘 준수하고 있다.

## 참고
https://mangkyu.tistory.com/147
