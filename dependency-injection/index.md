# Dependency Injection


## Dependency Injection
Dependency Injection은 말 그대로 의존성 주입을 말한다. 이는 **객체 간의 의존성을 외부에서 주입하여 관리**하겠다라는 개념이다. 의존성이 높으면 코드의 재사용성이 떨어지고 변경에 유연하지 못하며 테스트 코드를 작성하기 어려워진다.

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

```java
private MemberService memberService;

@Autowired
public xxxController(MemberService memberService){ ... }
```

필드가 단 하나뿐이라면 @Autowired 애노테이션이 없어도  자동 주입을 해준다.

## 왜 Constructor Injection을 써야 하는가?
기존에 가장 흔히 사용하는 Field Injection이 아니라 Constructor Injection을 써야 하는 이유는 다음과 같다.

### 1. 불변
대부분의 의존관계 주입은 한번 일어나면 애플리케이션 종료시점까지 의존관계를 변경할 일이 없다. 오히려 대부분의 의존관계는 애플리케이션 종료 전까지 변하면 안된다.(불변해야 한다.)

수정자 주입을 사용하면, setXxx 메서드를 public으로 열어두어야 한다. 누군가 실수로 변경할 수 도 있고, 변경하면 안되는 메서드를 열어두는 것은 좋은 설계 방법이 아니다.
**생성자 주입은 객체를 생성할 때 딱 1번만 호출되므로 이후에 호출되는 일이 없다. 따라서 불변하게 설계할 수 있다.**

### 2. 누락 방지
생성자 주입을 사용하면 **주입 데이터를 누락 했을 때 컴파일 오류가 발생한다.** 그리고 IDE에서 바로 어떤 값을 필수로 주입해야 하는지 알 수 있다.

### 3. final 키워드
생성자 주입을 사용하면 필드에 final 키워드를 사용할 수 있다. 그래서 **생성자에서 혹시라도 값이 설정되지 않는 오류를 컴파일 시점에 막아준다.**

{{< admonition note "정리" true >}}
* 생성자 주입 방식을 선택하는 이유는 여러가지가 있지만, 프레임워크에 의존하지 않고, 순수한 자바 언어의 특징을 잘 살리는 방법이기도 하다.
* 기본으로 생성자 주입을 사용하고, 필수 값이 아닌 경우에는 수정자 주입 방식을 옵션으로 부여하면 된다.
* 생성자 주입과 수정자 주입을 동시에 사용할 수 있다.
* 항상 생성자 주입을 선택해라! 그리고 가끔 옵션이 필요하면 수정자 주입을 선택해라. 필드 주입은 사용하지
않는게 좋다.
{{< /admonition >}}

## 참고
https://tecoble.techcourse.co.kr/post/2020-07-18-di-constuctor-injection/

https://jwchung.github.io/DI%EB%8A%94-IoC%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EC%95%8A%EC%95%84%EB%8F%84-%EB%90%9C%EB%8B%A4

https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8

