# Lombok 어노테이션


## 생성자 자동 생성
Lombok을 사용하면 생성자를 자동으로 생성할 수 있다.

- **@NoArgsConstructor** 어노테이션 : 파라미터가 없는 기본 생성자를 생성해준다.
- **@AllArgsConstructor** 어노테이션 : 모든 필드 값을 파라미터로 받는 생성자를 만들어준다.
- **@RequiredArgsConstructor** 어노테이션 : final이나 @NonNull인 필드 값만 파라미터로 받는 생성자를 만들어준다.
  - 주로 의존성 주입(Dependency Injection) 편의성을 위해서 사용된다.

{{< admonition note "@RequiredArgsConstructor 어노테이션" true >}}
@RequiredArgsConstructor 어노테이션은 스프링 의존성 주입의 특징 중 한가지를 이용하는데 이는 다음과 같다.

어떠한 빈(Bean)에 생성자가 오직 하나만 있고, 생성자의 파라미터 타입이 빈으로 등록 가능한 존재라면 이 빈은 @Autowired 어노테이션 없이도 의존성 주입이 가능하다.
{{< /admonition >}}

## 참고
https://www.daleseo.com/lombok-popular-annotations/

https://medium.com/webeveloper/requiredargsconstructor-%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85-dependency-injection-4f1b0ac33561

