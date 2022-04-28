# Dependency Injection


## Dependency Injection
Dependency Injection은 말 그대로 의존성 주입을 말한다. 이는 **객체 간의 의존성을 외부에서 주입하여 관리**하겠다라는 개념이다.

의존성이 높으면 코드의 재사용성이 떨어지고 변경에 유연하지 못하며 테스트 코드를 작성하기 어려워진다.

외부에서 의존을 주입받으면 의존을 내부에서 정의하지 않기 때문에 객체 간의 의존성을 줄여주고 코드의 재사용성도 증가하며 변화에 민감하지 않을 수 있다. 이때 변화에 민감하다는 말은 객체 자신이 아니라 의존하고 있는 다른 객체의 변경으로부터 민감한 정도를 말한다. 의존의 정도가 작을수록 의존 객체의 변경에 크게 영향을 받지 않는다.

외부에서 의존을 주입받는 방법은 기존에 마치 이전에 내부에서 의존을 관리하는 방법을 뒤바꾼듯한 모양이다. 이를 `IoC(Inversion of Control)`이라 한다.

{{< admonition note "DI vs IoC" true >}}
IoC는 객체 생명 관리, 흐름 제어를 제 3자에게 위임하는 프로그래밍 모델이다.

DI라는 말은 마틴파울러와 그 주변사람들이 객체 주입이라는 의미를 명확히 하기 위해 만들어졌다.
**컨테이너차원에서 수많은 객체들에 의존관계를 파악하고 런타임시점에서 다이나믹하게 객체를 주임하여 유연한 프로그래밍을 할 수 있도록 하는 패턴**에 더 명확한 이름을 부여하기 위해 토론 끝에 Dependency Injection 이라는 용어를 만들었다.
{{< /admonition >}}

## Dependency Injection의 방법
### Field Injection
가장 흔히 볼 수 있는 Injection 방법으로 사용하기도 간편하고 코드도 읽기 쉽다.
```java
@Service
public class LineService {
    @Autowired
    private LineRepository repository;

    public LineService() {
    }
}
```

### Setter Injection
이 방식은 Setter/Getter와 같은 접근자 메서드(Accessor)를 최소한으로 해야한다는 원칙과도 어긋나고, 주입받는 객체가 변경될 가능성도 거의 없기에 거의 사용하지 않는다. 

```java
@Service
public class LineService {
    private LineRepository repository;

    public LineService() {
    }

    @Autowired
    public void setRepository(LineRepository lineRepository) {
        this.repository = lineRepository;
    }
}
```

### Constructor Injection
필드가 단 하나뿐이라면 @Autowired 애노테이션이 없어도  자동 주입을 해준다.

클래스에 컴포넌트 애노테이션( @Controller, @Service 등도 내부적으로 @Component를 포함하는 메타 애노테이션이다.) 이 있는경우 Bean 등록을 하기 위해서 객체를 생성하는데 final 키워드가 있는 멤버필드가 있다면 생성시점에서 해당 필드들을 모두 주입 해 줘야 하기에 @Autowired 애노테이션이 없더라도 해당 컴포넌트들을 찾아서 주입 해 준다. 

```java
@Service
public class LineService {
    private final LineRepository repository;

    public LineService(LineRepository repository) {
        this.repository = repository;
    }
}
```

## 왜 Constructor Injection을 써야 하는가?
기존에 가장 흔히 사용하는 Field Injection이 아니라 Constructor Injection을 써야 하는 이유는 무엇일까?

### 1. 단일 책임의 원칙.
Field Injection은 의존성 주입이 너무 쉬우므로 무분별한 의존성을 주입할 수도 있다. 그러면 하나의 클래스에서 지나치게 많은 기능을 하게 될 수 있는 여지가 있고, ‘객체는 그에 맞는 동작만 한다’라는 단일 책임의 원칙이 깨지기 쉽다.

Constructor Injection을 사용하면 의존성을 주입해야 하는 대상이 많아질수록 생성자의 인자가 늘어난다. 이는 의존관계의 복잡성을 쉽게 파악할 수 있도록 도와주므로 리팩토링의 실마리를 제공한다.

### 2. DI Container와의 낮은 결합도
Field Injection을 사용하면 의존 클래스를 곧바로 인스턴스화 시킬 수 없다. 만약 DI Container 밖의 환경에서 의존성을 주입받는 클래스의 객체를 참조할 때, Dependency를 정의해두는 Reflection을 사용하는 방법 외에는 참조할 방법이 없다. 생성자 또는 Setter가 존재하지 않는다면 의존 객체 필드를 설정할 수 있는 방법이 없기 때문이다.

Constructor Injection은 생성자로 의존성을 주입받기 때문에 DI Container에 의존하지 않고 사용할 수 있고, 그 덕분에 테스트에서도 더 용이함을 보인다.

### 3. 필드의 불변성 보장
Field Injection은 객체를 생성하고 의존성을 Reflection으로 주입받기 때문에 필드 변수를 Immutable로 선언할 수 없다.

Constructor Injection은 필드를 final로 선언할 수 있기 때문에 필드의 변경에 대해 안전하다. 이는 객체의 변경에 따른 비용을 절약할 수 있도록 도와준다.

### 순환 의존 방지
Constructor Injection에서는 순환 의존성을 가질 경우 BeanCurrentlyInCreationException이 발생해서 문제 상황을 알 수 있게 해준다.

## 참고
https://tecoble.techcourse.co.kr/post/2020-07-18-di-constuctor-injection/

https://jwchung.github.io/DI%EB%8A%94-IoC%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EC%95%8A%EC%95%84%EB%8F%84-%EB%90%9C%EB%8B%A4

