---
title: HTTP
date: 2023-09-06T15:12:26+09:00
categories:
  - web
tags: 
  - HTTP
---

## HTTP

HTTP는 HyperText Transfer Protocol의 약자로, HTML, text, image, 음성, 영상, 파일, JSON, XML 등 **거의 모든 형태의 데이터를 전송 가능한 프로토콜**이다.

## HTTP 특징

### 클라이언트-서버 구조

클라이언트는 서버에 요청을 보내고, 응답을 대기한다. 서버는 요청에 대한 결과를 만들어서 응답한다.

### 무상태 프로토콜(Stateless))

서버가 클라이언트의 상태를 보존하지 않는다.

- 장점: 서버 확장성 높음(스케일 아웃)
- 단점: 클라이언트가 추가 데이터 전송

모든 것을 무상태로 설계 할 수 없는 경우에는 브라우저 쿠키와 서버 세션등을 사용해서 상태 유지를 함으로써 무상태 프로토콜의 한계를 극복할 수 있다.

### 비연결성(Connectionless)

HTTP는 연결을 유지하지 않는 모델이다. 일반적으로 초 단위의 이하의 빠른 속도로 응답하므로, 서버 자원을 매우 효율적으로 사용할 수 있다.

## 참고

https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC
