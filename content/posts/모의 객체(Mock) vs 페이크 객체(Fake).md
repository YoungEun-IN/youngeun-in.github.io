---
title: 페이크 객체(Fake)
date: 2022-01-01T15:12:26+09:00
categories:
  - 설계방법론
tags: 
  - mock
  - fake

---
## 페이크 객체

**페이크 객체(Fake Object)란 여러 상태를 대표할 수 있도록 구현된 객체로, 실제 로직이 구현된 것 처럼 보이게 한다.** 실 DB에 접속해 동일한 모양이 보이도록 객체 내부에 구현할 수 있다.

모의 객체 대신 페이크 객체를 사용해야 하는 이유는 다음과 같다.

|Mock|Fake|
|---|---|
|코드가 장황해지고 이해가 어려워진다.|코드가 간결해지고 유지보수성이 좋아진다.|
|가정 / 결과를 통해 구현하기 때문에 내부 구현을 알아야한다. | 클래스 내부의 구현에 대해 알지 못해도 된다. |
|내부 구현을 변경했을 때 테스트에서 실패한다. | 내부 구현을 변경했을 때 테스트에서 실패하지 않는다.|
|객체 간 강한 결합력을 만든다.|객체지향적인 설계에 관해 고민하도록 도와준다.|

## 페이크 객체 사용

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

//인터페이스 선언
interface GitHubService {
  GitHub connect() throws IOException;
}
```

```java
import org.kohsuke.github.GitHubBuilder;
import org.kohsuke.github.GitHub;
import java.io.IOException;

//인터페이스를 구현한 클래스 생성하고 메소드를 오버라이드한다.
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

  //서비스를 주입받는다.
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
