---
title: WebFlux, WebClient
date: 2024-03-05T15:12:26+09:00
categories:
  - spring
tags: 
  - WebFlux
  - WebClient
---

## 리액티브 시스템
**리액티브 시스템이란 클라이언트 요청에 바로 응답을 해주는 시스템을 말한다.**

리액티브 시스템의 특징은 다음과 같다.
- 응답성이 높다.
- 탄력적이고 유연하다.
- 메시지 기반으로 통신한다.

## 리액티브 프로그래밍
**리액티브 프로그래밍이란 리액티브 시스템을 구축하는 데 필요한 프로그래밍 모델을 말한다.**

리액티브 프로그래밍의 특징은 다음과 같다.

- 데이터 소스에 변경이 있을 때마다 데이터를 전파
- 선언형 프로그래밍 패러다임 : 실행할 동작을 구체적으로 명시하지 않고 목표만 정의함
- 함수형 프로그래밍 기법 사용

## 리액티브 스트림즈(Reactive Streams)

리액티브 스트림즈란 리액티브 프로그래밍을 표준화한 명세를 말한다.

리액티브 스트림즈의 구현체는 다음과 같다.
- RxJava
- Java 9 Flow API
- Akka Streams
- Reactor

## Spring Webflux

**Spring Webflux란 Spring 5부터 지원하는 리액티브 웹 프레임워크이다.** 비동기 Non-Blocking I/O 방식으로 적은 수의 쓰레드를 사용한다. Spring Webflux는 Reactive Streams의 구현체 중에 하나인 Reactor에 의존하여 비동기 로직을 구성하고 리액티브 스트림을 제공한다.

{{< admonition note "리액터(Reactor)" true >}}
**리액터(Reactor)란 리액티브 프로그래밍을 위한 리액티브 라이브러리이다.** Reactive Streams 스펙을 구현한 구현체 중 하나이다. Spring 에코 시스템에서 Reactive Stack의 기반이 되며 Spring WebFlux 프레임워크에 포함이 되어 있다.
{{< /admonition >}}

Spring Webflux는 적은 수의 스레드를 사용하여 여러 클라이언트의 요청을 처리할수 있고, 자원(메모리,CPU)를 효율적으로 사용할수 있다.

Spring WebFlux를 사용하기 적합한 시스템은 다음과 같다.
* Blocking I/O 방식으로 처리하는데 한계가 있는 대량의 요청 트래픽이 발생하는 시스템
* 마이크로 서비스 기반 시스템
* 스트리밍 시스템 또는 실시간 시스템
* 네트워크 접속이 느린 클라이언트의 요청 처리가 필요한 시스템

## Spring WebClient

**WebClient는 Spring WebFlux에서 HTTP Client로 사용되는 비동기적으로 작동하는 모듈이다.** 내부적으로 WebClient는 HTTP Client 라이브러리에 위임하는데 디폴트로 Netty의 Http Client를 사용한다.

### Spring WebClient 사용

build.gradle 파일에서 필요한 의존성을 추가한다.

```groovy
plugins {
    id 'org.springframework.boot' version '2.5.4'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-webflux'
    implementation 'org.springframework.boot:spring-boot-starter-reactor-netty'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'io.projectreactor:reactor-test'
}

test {
    useJUnitPlatform()
}
```

WebClient를 사용하여 외부 API에 비동기 요청을 보내는 서비스를 만든다.

```java
package com.example.demo.service;

import org.springframework.stereotype.Service;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

@Service
public class WebClientService {

    private final WebClient webClient;

    public WebClientService(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder.baseUrl("https://jsonplaceholder.typicode.com").build();
    }

    public Mono<String> getPostById(int id) {
        return this.webClient.get()
                .uri("/posts/{id}", id)
                .retrieve()
                .bodyToMono(String.class);
    }
}
```

컨트롤러를 만들어 WebClientService를 사용하여 비동기 요청을 처리하는 엔드포인트를 추가한다.

```java
package com.example.demo.controller;

import com.example.demo.service.WebClientService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Mono;

@RestController
public class WebClientController {

    private final WebClientService webClientService;

    public WebClientController(WebClientService webClientService) {
        this.webClientService = webClientService;
    }

    @GetMapping("/posts/{id}")
    public Mono<String> getPostById(@PathVariable int id) {
        return webClientService.getPostById(id);
    }
}
```

## 참고
https://velog.io/@rnqhstlr2297/Spring-Webflux%EC%99%80-WebClient

https://www.inflearn.com/course/spring-reactive-web-application-reactor1%EB%B6%80
