---
title: ETag
date: 2022-01-06T15:12:26+09:00
categories:
  - web
tags: 
  - http-cache
  - etag
---

ETag(entity tag)는 **웹 서버가 주어진 URL의 콘텐츠가 변경되었는지 알려주고 이를 반환하는 HTTP 응답 헤더**이다.

캐시를 사용하면 불필요한 요청을 줄이면서 서버의 부하를 줄일 수 있고, 미리 캐시에 저장해 놓은 값을 사용함으로써 빠른 응답을 할 수 있다. ETag는 **사용하는 캐시가 유효한지 검증하기 위해 사용**한다. Static file(js, css, image) 뿐만 아니라 API와 같은 Dynamic content에도 간단하게 Cache기능을 사용하게 설정하면 API속도도 증가되고 유저 입장에서는 네트웍 트래픽을 줄일 수 있다. API와 같이 언제 바뀔지 모르는 데이터의 Cache는 쉽지 않은데, Etag를 이용하면 서버에서 새로운 데이터를 먼저 확인하고 줄 수 있기 때문에 API에도 충분히 적용가능하다.

## ETag를 사용한 요청과 응답 예시

1. 클라이언트가 요청을 보낸다.

```http
curl -H "Accept: application/json" 
     -i http://localhost:8080/spring-boot-rest/foos/1
```

2. 서버는 `ETag`를 응답 header에 담아서 보낸다.

```http
HTTP/1.1 200 OK
ETag: "f88dd058fe004909615a64f01be66a7"
Content-Type: application/json;charset=UTF-8
Content-Length: 52
```

3. 클라이언트는 재요청할 때 `ETag`를 header의 `If-None-Match`에 담아 요청을 보낸다. 
{{< admonition >}}
ETag를 사용할 때 Conditional headers로  `If-None-Match`와 `If-Match`가 있다. 
- [If-None-Match](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match) - 클라이언트에서 캐싱된 ETag와 서버의 ETag가 다를 때 요청을 처리한다.
- [If-Match](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Match)  - 클라이언트에서 캐싱된 ETag와 서버의 ETag가 같을 때 요청을 처리한다.
{{</ admonition >}}

4. 클라이언트는 `If-None-Match`헤더를 포함하여 서버에 요청을 다시 한다.

```http
curl -H "Accept: application/json" 
     -H 'If-None-Match: "f88dd058fe004909615a64f01be66a7"'
     -i http://localhost:8080/spring-boot-rest/foos/1
```

5. 리소스가 바뀌지 않았기 때문에 서버는 `304 Not Modified`를 응답한다. `ETag`는 이전 요청에 대한 응답과 같다.

```http
HTTP/1.1 304 Not Modified
ETag: "f88dd058fe004909615a64f01be66a7"
```

6. 클라이언트의 요청에 대해 다른 응답을 하도록 서버의 데이터를 바꾼다.

7. 클라이언트는 같은 요청을 다시 한다. 요청을 다시 할 때는 마지막으로 가지고 있던 ETag를 담아서 보낼 것이다. 

```http
curl -H "Accept: application/json" 
     -H 'If-None-Match: "f88dd058fe004909615a64f01be66a7"' 
     -i http://localhost:8080/spring-boot-rest/foos/1
```

8. 클라이언트에서 보낸 ETag와 서버의 ETag가 다르기 때문에 서버는 요청을 처리한다. 리소스가 바뀌었으니 새로운 ETag를 header에 담아 보낸다. 새로운 요청을 처리했기 때문에 서버는 `200 OK`를 응답한다.

```http
HTTP/1.1 200 OK
ETag: "03cb37ca667706c68c0aad4cb04c3a211"
Content-Type: application/json;charset=UTF-8
Content-Length: 56
```

## ETag를 사용하지 않은 API vs 사용한 API

ETag 사용 예시와 ETag를 사용한 API와 사용하지 않은 API를 비교를 설명하기 위해 간단하게 Controller를 작성해 보았다.

```java
@RequestMapping("/posts")
@RestController
public class PostController {

    // ...
    
    @GetMapping("/no-etag")
    public ResponseEntity<List<PostResponse>> findAllWhenNoETag() {
        return ResponseEntity.ok().body(postService.findAll());
    }
    
    @GetMapping("/etag")
    public ResponseEntity<List<PostResponse>> findAllWhenETag() {
        return ResponseEntity.ok().body(postService.findAll());
    }
    
    // ...
}
```

그리고 ETag 설정으로 ShallowEtagHeaderFilter를 Bean으로 등록해준다.

```java
@Configuration
public class ETagHeaderFilter {

    @Bean
    public ShallowEtagHeaderFilter shallowEtagHeaderFilter() {
        return new ShallowEtagHeaderFilter();
    }
}
```

추가 필터를 구성할 필요 없다면 위의 코드와 같이 작성해도 된다. 하지만 ETag를 사용한 API와 사용하지 않은 API를 비교하기 위해 필터를 사용했다.

추가 필터 구성을 하고 싶다면 다음과 같이 설정해주면 된다.

```java
@Configuration
public class ETagHeaderFilter {

    @Bean
    public FilterRegistrationBean<ShallowEtagHeaderFilter> shallowEtagHeaderFilter() {
        FilterRegistrationBean<ShallowEtagHeaderFilter> filterRegistrationBean
                = new FilterRegistrationBean<>( new ShallowEtagHeaderFilter());
        filterRegistrationBean.addUrlPatterns("/posts/etag");
        filterRegistrationBean.setName("PostAPIFilter");
        return filterRegistrationBean;
    }
}
```

현재 PostController에서 `/posts/etag`만  etag를 사용한다는 설정이다. 만약 `/post/`에 대해 전부 ETag를 설정하고 싶다면  `filterRegistrationBean.addUrlPatterns("/posts/*")` 이렇게 설정하면 된다.

그럼 이제 `/no-etag`와 `/etag`를 호출해 보자.

![image](https://user-images.githubusercontent.com/45934117/94986209-cb10ab80-0597-11eb-9b8d-d88597fcc56e.png)

얼핏 보면 둘의 차이가 안 보인다. 하지만 Response Headers를 보면 차이를 볼 수 있다.

![image](https://user-images.githubusercontent.com/45934117/94986113-e929dc00-0596-11eb-84c1-7f12b318c509.png)

두 응답의 차이를 볼 수 있는 곳은 ETag일 것이다. `/etag`는 ETag를 사용하고 있기 때문에 응답으로 ETag를 header에 해시값으로 보내준다. 이는 재요청할 때 header의 `If-None-Match`의 값으로 보내 줄 것이다.

```http
If-None-Match: "0fad8e1b47f45fa4ce7fef400e87c9289"
```

이렇게 ETag를 `/etag` 요청 header의 `If-None-Match`에 담아 재요청해 보겠다.

![image](https://user-images.githubusercontent.com/45934117/94986192-af0d0a00-0597-11eb-8966-f7123a1fd879.png)

`etag`를 보면 앞서 설명했듯이 같은 요청에 대해서 304 상태 코드를 응답한다. 이는 서버에서 캐시 유효성 검사를 한 결과 변경되지 않았기 때문이다. 

여기서 봐야 할 것은 사이즈다. `no-etag`는 재요청에 대해서 `796B -> 796B`인 반면에 `etag`는 `820B -> 145B`이다. 이유는 ETag를 사용하지 않으면 했던 일을 똑같이 또 하지만, ETag를 사용하면 같은 요청에 대해서 변경된 리소스가 없다면 304 상태 코드와 ETag를 header에 담아 보내줄 뿐 요청에 대한 리소스를 또 보내지 않는다.

## 참고
https://tecoble.techcourse.co.kr/post/2020-09-30-ETag-with-Spring/
