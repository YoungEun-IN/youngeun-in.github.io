---
title: Etag
date: 2022-02-27T15:12:26+09:00
categories:
  - network
tags: 
  - http-cache
---
## HTTP 캐시
Static file(js, css, image) 뿐만 아니라 API와 같은 Dynamic content에도 간단하게 Cache기능을 사용하게 설정하면 API속도도 증가되고 유저 입장에서는 네트웍 트래픽을 줄일 수 있다.
API와 같이 언제 바뀔지 모르는 데이터의 Cache는 쉽지 않은데, Etag를 이용하면 서버에서 새로운 데이터를 먼저 확인하고 줄 수 있기 때문에 API에도 충분히 적용가능하다.

## Etag
API데이터의 MD5 Hash를 ETag로 사용하게 되면 DB부하를 줄일 순 없지만, Response 시간을 줄이는데 도움이 될 수 있다.
Response에 다음과 같은 Etag 헤더를 추가해주면, 다음 동일한 API를 부르면 아래와 같은 Request 헤더에 체크하는 헤더가 추가된다.
```
If-None-Match: "15f0fff99ed5aae4edffdd6496d7131f"
```

서버에서 같은 ETag를 가지고 있다면 Http code 304 를 리턴해주면 된다.

## Etag 작동 순서
1. API 핸들러 처리(DB query)
2. 결과 Json에 MD5 Hash
3. If-None-Match 헤더에 있는 값과 hash값을 비교
4. 만약 동일하다면 304로 리턴
5. 다르다면 200에 결과 Json리턴
