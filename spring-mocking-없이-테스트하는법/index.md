# Spring mocking 없이 테스트하는법

## Spring mocking 없이 테스트하려면?
아래 코드를 mocking 없이 테스트하려면 어떻게 해야 할까?

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

Repository 인스턴스를 생성 시에 외부에서 생성한 GitHubService 인스턴스를 주입하면 된다.
설명과 코드는 다음과 같다.

```java
package com.example.demo;

import org.kohsuke.github.GHIssueState;
import org.kohsuke.github.GHRepository;
import org.kohsuke.github.GitHub;
import org.kohsuke.github.GitHubBuilder;

import java.io.IOException;

public class RepositoryRank {
    //인터페이스 선언
    interface GitHubService {
        GitHub connect() throws IOException;
    }

    //인터페이스를 구현한 클래스 생성하고 모킹하고자 하는 메소드를 오버라이드한다.
    static class DefaultGitHubService implements GitHubService {

        @Override
        public GitHub connect() throws IOException {
            return new GitHubBuilder().build();
        }
    }

    private final GitHubService gitHubService;

    //서비스를 주입받는다.
    public RepositoryRank(GitHubService gitHubService) {
        this.gitHubService = gitHubService;
    }

    public int getPoint(String repositoryName) throws IOException {
        GitHub github = gitHubService.connect();
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

    public void main(String[] args) throws IOException {
        GitHubService gitHubService = new DefaultGitHubService();
        //Repository 인스턴스를 생성 시에 외부에서 생성한 GitHubService 인스턴스를 주입
        RepositoryRank repositoryRank = new RepositoryRank(gitHubService);
        int point = repositoryRank.getPoint("whiteship/live-study");
        System.out.println(point);
    }
}
```

## 참고
백기선님의 유튜브를 참고했다.
https://youtu.be/bJfbPWEMj_c

