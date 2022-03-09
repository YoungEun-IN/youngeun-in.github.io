---
title: Dependency Inversion Principle
date: 2022-03-09T14:12:26+09:00
categories:
  - 개발방법론
tags: 
  - solid
---

## DIP(Dependency Inversion Principle)

우리가 다루는 모듈은 고수준 모듈과 저수준 모듈로 나눌 수 있다. 고수준 모듈이란 의미있는 단일 기능을 제공하는 모듈이며, 저수준 모듈은 고수준 모듈의 기능을 구현하기 위해 필요한 하위 기능의 실제 구현인 모듈이다. Layered Architecture 상에서 Application 및 Domain 등의 고수준 모듈은 Infrastructure라는 저수준 모듈을 의존한다. 그 결과, 구현 부분의 변경에 유연하지 못하고 테스트하기 어렵다는 문제점이 발생한다.

DIP(Dependency Inversion Principle)이란 의존 관계를 역전시켜서 저수준 모듈이 고수준 모듈에 의존하도록 구현하는 것을 의미한다. 고수준 모듈이 저수준 모듈을 직접 의존하는 것이 아니라, **저수준 모듈이 인터페이스 등 추상을 매개체로 고수준 모듈을 참조**하도록 한다. 이를 통해 구현 기술과 관련된 종속성을 쉽게 제거할 수 있다.

## DIP 예제

```java
public interface PlatformContributionCalculator {

    Contribution calculate(String accessToken, String username);
}
```

```java
@RequiredArgsConstructor
@Component
public class GithubContributionCalculator implements PlatformContributionCalculator {
    @Value("${github.contribution.url}")
    private final String url;

    public Contribution calculate(String accessToken, String username) {
        HttpHeaders httpHeaders = new HttpHeaders();
        httpHeaders.setBearerAuth(accessToken);
        RequestEntity<Void> requestEntity = RequestEntity
            .get(url)
            .headers(httpHeaders)
            .build();
        RestTemplate restTemplate = new RestTemplate();
        return restTemplate.exchange(requestEntity, Contribution.class)
            .getBody();
    }
}
```

```java
@RequiredArgsConstructor
@Transactional
@Service
public class UserService {

    private final UserRepository userRepository;
    private final PlatformContributionCalculator platformContributionCalculator;

    public ContributionResponseDto calculateContributions(ContributionRequestDto requestDto) {
        User user = userRepository.findByName(requestDto.getUsername())
            .orElseThrow(InvalidUserException::new);

        Contribution contribution = platformContributionCalculator
            .calculate(requestDto.getAccessToken(), user.getName());

        return ContributionResponseDto.builder()
            .starsCount(contribution.getStarsCount())
            .commitsCount(contribution.getCommitsCount())
            .prsCount(contribution.getPrsCount())
            .issuesCount(contribution.getIssuesCount())
            .reposCount(contribution.getReposCount())
            .build();
    }
}
```

Application 계층이 Infrastructure 계층의 구현체가 아닌 Domain 계층의 `PlatformContributionCalculator` 인터페이스를 의존하도록 구현한다. 만약 요구사항 변경으로 인해 저수준 모듈이 GitHub 플랫폼 연동에서 GitLab 플랫폼 연동으로 변경되더라도, 고수준 모듈에서의 변경을 최소화할 수 있게 된다.

---
```java
@Autowired
private UserRepository userRepository;

private UserService userService;

@BeforeEach
void setUp() {
    PlatformContributionCalculator platformContributionCalculator = (accessToken, username) -> {
        // do something
    };
    userService = new UserService(userRepository, platformContributionCalculator);        
}
```
테스트를 진행할 때 `PlatformContributionCalculator`에 적합한 Mock Object를 주입할 수 있게 된다. 따라서 GitHub 서버가 아닌 Mock 서버로 API 요청을 보내게끔 테스트 대역을 조절함으로써, Access Token으로 인한 문제에서 벗어나 자동화된 테스트를 쉽게 작성할 수 있다.

---
```java
@TestConfiguration
public class InfrastructureTestConfiguration {

    @Bean
    public PlatformContributionCalculator platformContributionCalculator() {
        return new MockContributionCalculator();
    }
}
```
테스트 클래스에 정의하는 것이 번거롭다면 대역을 테스트용 Bean으로 주입해 여러 통합 테스트에서 사용할 수 있다.

## 참고
https://tecoble.techcourse.co.kr/post/2021-11-21-dip/
