# 모의 객체(Mock) vs 페이크 객체(Fake)

## 모의 객체(Mock) vs 페이크 객체(Fake)

페이크 객체는 interface의 일부이며 인터페이스와 함께 제공된다.

```java
interface Exchange {
    float rate(String origin, String target);
    final class Fake implements Exchange {
        @Override
        float rate(String origin, String target) {
            return 1.2345;
        }
    }
}
```
모의 객체 대신 페이크 객체를 사용해야 하는 이유는 다음과 같다.

|Mock|Fake|
|---|---|
|코드가 장황해지고 이해가 어려워진다.|코드가 간결해지고 유지보수성이 좋아진다.|
|가정 / 결과를 통해 구현하기 때문에 내부 구현을 알아야한다. | 클래스 내부의 구현에 대해 알지 못해도 된다. |
|내부 구현을 변경했을 때 테스트에서 실패한다. | 내부 구현을 변경했을 때 테스트에서 실패하지 않는다.|
|객체 간 강한 결합력을 만든다.|객체지향적인 설계에 관해 고민하도록 도와준다.|

## Fake 객체 사용

```java
package com.example.demo;

import org.kohsuke.github.GHIssueState;
import org.kohsuke.github.GHRepository;
import org.kohsuke.github.GitHub;

import java.io.IOException;

public class RepositoryRank {
    public int getPoint(String repositoryName) throws IOException {
        GitHub github = GitHub.connect();
        GHRepository repository = github.getRepository(repositoryName);

        int points = 0;
        if (repository.hasIssues()) {
            points += 1;
        }

        if (repository.getPullRequests(GHIssueState.CLOSED).size() > 0) {
            points += 1;
        }

        points += repository.getStargazersCount();
        points += repository.getForksCount();

        return points;
    }

    public static void main(String[] args) throws IOException {
        RepositoryRank repositoryRank = new RepositoryRank();
        int point = repositoryRank.getPoint("whiteship/live-study");
        System.out.println(point);

    }
}
```

위 코드를 Fake 객체를 사용하여 테스트하려면 Repository 인스턴스를 생성 시에 외부에서 생성한 GitHubService 인스턴스를 주입하면 된다.

설명과 코드는 다음과 같다.

```java
import java.io.IOException;

interface GitHubService {
  GitHub connect() throws IOException;
}
```

```java
import org.kohsuke.github.GitHubBuilder;
import org.kohsuke.github.GitHub;
import java.io.IOException;

public class DefaultGitHubService implements GitHubService {
  @Override
  public GitHub connect() throws IOException {
    return GitHub.connect();
  }
}
```

```java
import org.kohsuke.github.GHRepository;
import org.kohsuke.github.GHIssueState;
import java.io.IOException;

public class RepositoryRank {
  private final GitHubService gitHubService;
  
  public RepositoryRank(final GitHubService gitHubService) {
    this.gitHubService = gitHubService;
  }

  public int getPoint(String repositoryName) throws IOException {
    GitHub github = gitHubService.connect();
    GHRepository repository = github.getRepository(repositoryName);
    
    // 생략
  }
}
```

## 참고
https://jessyt.tistory.com/152

https://github.com/seovalue/elegant_object/issues/13

https://youtu.be/bJfbPWEMj_c

