---
title: HTTPS
date: 2022-04-15T15:12:26+09:00
categories:
  - web
tags: 
  - https
---

HTTP와 HTTPS의 가장 큰 차이점은 SSL 인증서이다. **SSL인증서는 클라이언트와 서버간의 통신을 공인된 제 3차 업체(CA)가 보증해주는 전자화된 문서**이다. SSL 인증서는 사용자가 사이트에서 제공하는 정보를 암호화한다. 암호화되어 전송되는 데이터는 중간에 누가 훔치거나 조작하려해도 암호화 되어있어서 해독할 수 없다. 

HTTPS 방식은 공개키 암호화 방식과 대칭키 암호화 방식의 장점을 활용해 함께 사용한다. **데이터를 대칭키 방식으로 암복호화하고, 공개키 방식으로 대칭키를 전달한다.**

## HTTPS의 동작과정

![image](https://user-images.githubusercontent.com/46465928/163977285-499e0513-6a79-45f3-8fb6-63a1468b0962.png)

### 클라이언트가 서버 접속하여 Handshaking 과정에서 서로 탐색
1. Client Hello : 클라이언트가 서버에게 전송할 데이터
    - 클라이언트 측에서 생성한 랜덤 데이터
    - 클라이언트가 사용할 수 있는 암호화 방식
    - 이전에 이미 Handshaking 기록이 있다면 자원 절약을 위해 기존 세션을 재활용하기 위한 세션 아이디

2. Server Hello : Client Hello에 대한 응답으로 전송할 데이터
    - 서버 측에서 생성한 랜덤 데이터
    - 서버가 선택한 클라이언트의 암호화 방식
    - SSL 인증서

3. Client 인증 확인
    - 서버로부터 받은 인증서가 CA에 의해 발급되었는지 본인이 가지고 있는 목록에서 확인하고, 목록에 있다면 CA 공개키로 인증서 복호화
    - 클라이언트, 서버 각각의 랜덤 데이터를 조합하여 pre master secret 값 생성(데이터 송수신 시 대칭키 암호화에 사용할 키)
    - pre master secret 값을 공개키 방식으로 서버 전달(공개키는 서버로부터 받은 인증서에 포함)

4. Server 인증 확인
    - 서버는 비공개키로 복호화하여 pre master secret 값 취득(대칭키 공유 완료)

5. Handshaking 종료

### 데이터 전송
  - 서버와 클라이언트는 session key를 활용해 데이터를 암복호화하여 데이터 송수신

### 연결 종료 및 session key 폐기

## 참고
https://velog.io/@averycode/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-HTTP%EC%99%80-HTTPS-%EB%8F%99%EC%9E%91-%EA%B3%BC%EC%A0%95
