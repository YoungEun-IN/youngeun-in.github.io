---
title: WebFlux, WebClient
date: 2024-03-05T15:12:26+09:00
categories:
  - spring
tags: 
  - WebFlux
  - WebClient
---

## 리액터(Reactor)
**리액터(Reactor)란 리액티브 프로그래밍을 위한 리액티브 라이브러리이다.** Reactive Streams 스펙을 구현한 구현체 중 하나이다. Spring 에코 시스템에서 Reactive Stack의 기반이 되며 Spring WebFlux 프레임워크에 포함이 되어 있다.

## Spring Webflux

**Spring Webflux란 Spring 5부터 지원하는 리액티브 웹 프레임워크이다.** 비동기 Non-Blocking I/O 방식으로 적은 수의 쓰레드를 사용한다. Spring Webflux는 Reactive Streams의 구현체 중에 하나인 Reactor에 의존하여 비동기 로직을 구성하고 리액티브 스트림을 제공한다.

Spring Webflux는 Single-Thread와 Non-Blocking방식을 사용한다.

{{< admonition note "Blocking방식, Non-Blocking방식" true >}}
**Blocking방식이란 요청한 작업이 끝날때까지 다른 작업을 하지않고 끝날때까지 기다리는것이다.** 즉 하나의 스레드가 요청을 처리하기 위해 처리함수를 실행하면 제어권도 함께 넘겨 해당 함수가 끝이날때까지 다른 함수를 호출할수 없는것이다.

**Non-Blocking방식이란 요청한 작업이 수행되는 동안 다른 작업을 할수 있는 방식이다.** 즉 호출자가 함수를 호출할때 제어권을 호출자가 가지고 있어 다른 작업을 수행할수 있다.
{{< /admonition >}}

Spring Webflux는 적은 수의 스레드를 사용하여 여러 클라이언트의 요청을 처리할수 있고, 자원(메모리,CPU)를 효율적으로 사용할수 있다.

Spring WebFlux를 사용하기 적합한 시스템은 다음과 같다.
* Blocking I/O 방식으로 처리하는데 한계가 있는 대량의 요청 트래픽이 발생하는 시스템
* 마이크로 서비스 기반 시스템
* 스트리밍 시스템 또는 실시간 시스템
* 네트워크 접속이 느린 클라이언트의 요청 처리가 필요한 시스템

## Spring WebClient

**WebClient는 Spring WebFlux에서 HTTP Client로 사용되는 비동기적으로 작동하는 모듈이다.** 내부적으로 WebClient는 HTTP Client 라이브러리에 위임하는데 디폴트로 Netty의 Http Client를 사용한다.

## 참고
https://velog.io/@rnqhstlr2297/Spring-Webflux%EC%99%80-WebClient

https://www.inflearn.com/course/spring-reactive-web-application-reactor1%EB%B6%80
