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

**ETag는 사용하는 캐시가 유효한지 검증하기 위해 사용한다.** Static file(js, css, image) 뿐만 아니라 API와 같은 Dynamic content에도 간단하게 Cache기능을 사용하게 설정하면 API속도도 증가되고 유저 입장에서는 네트웍 트래픽을 줄일 수 있다. API와 같이 언제 바뀔지 모르는 데이터의 Cache는 쉽지 않은데, Etag를 이용하면 서버에서 새로운 데이터를 먼저 확인하고 줄 수 있기 때문에 API에도 충분히 적용가능하다.

## ETag를 사용했을 때 API 흐름

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

```http
curl -H "Accept: application/json" 
     -H 'If-None-Match: "f88dd058fe004909615a64f01be66a7"'
     -i http://localhost:8080/spring-boot-rest/foos/1
```

4. 리소스가 바뀌지 않았기 때문에 서버는 `304 Not Modified`를 응답한다. `ETag`는 이전 요청에 대한 응답과 같다.

```http
HTTP/1.1 304 Not Modified
ETag: "f88dd058fe004909615a64f01be66a7"
```

5. 클라이언트의 요청에 대해 다른 응답을 하도록 서버의 데이터를 바꾼다.

6. 클라이언트는 같은 요청을 다시 한다. 요청을 다시 할 때는 마지막으로 가지고 있던 ETag를 담아서 보낼 것이다. 

```http
curl -H "Accept: application/json" 
     -H 'If-None-Match: "f88dd058fe004909615a64f01be66a7"' 
     -i http://localhost:8080/spring-boot-rest/foos/1
```

7. 클라이언트에서 보낸 ETag와 서버의 ETag가 다르기 때문에 서버는 요청을 처리한다. 리소스가 바뀌었으니 새로운 ETag를 header에 담아 보낸다. 새로운 요청을 처리했기 때문에 서버는 `200 OK`를 응답한다.

```http
HTTP/1.1 200 OK
ETag: "03cb37ca667706c68c0aad4cb04c3a211"
Content-Type: application/json;charset=UTF-8
Content-Length: 56
```

## ETag 설정과 사용

Controller를 작성한다.

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

추가 필터를 구성할 필요 없다면 위의 코드와 같이 작성해도 된다. 추가 필터 구성을 하고 싶다면 다음과 같이 설정해주면 된다.

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

여기서 봐야 할 것은 사이즈다. `no-etag`는 재요청에 대해서 `796B -> 796B`인 반면에 `etag`는 `820B -> 145B`이다. 이유는 ETag를 사용하지 않으면 했던 일을 똑같이 또 하지만, ETag를 사용하면 같은 요청에 대해서 변경된 리소스가 없다면 304 상태 코드와 ETag를 header에 담아 보내줄 뿐 요청에 대한 리소스를 또 보내지 않기 때문이다.

## ETag 사용 시 고려사항

1. **충돌 가능성**: ETag는 고유한 식별자를 사용하지만, 매우 드물긴 해도 동일한 ETag 값을 가진 서로 다른 리소스가 존재할 가능성이 있다. 이 경우 캐시 일관성이 깨질 수 있다.
2. **성능 문제**: 서버가 파일의 콘텐츠, 메타데이터 등의 변화를 추적하여 ETag를 생성해야 하기 때문에 이 과정에서 추가적인 성능 부담이 발생할 수 있다. 특히 파일이 자주 변경되는 경우 이 부담은 더 커질 수 있다.
3. **보안 문제**: ETag가 파일의 해시값을 포함하는 경우, 이를 통해 파일의 변경 내역을 추적할 수 있어 보안 문제가 발생할 수 있다. 예를 들어, ETag 값을 통해 사용자의 행동을 추적하거나 캐시된 민감한 데이터가 노출될 수 있다.
4. **네트워크 비용**: ETag를 사용하여 불필요한 데이터를 전송하지 않을 수 있지만, ETag 자체를 주고받는 데에도 네트워크 비용이 발생합니다. 특히 ETag 값이 복잡하거나 큰 경우 이 비용이 증가할 수 있다.
5. **서버 간 일관성 문제**: 여러 대의 서버가 로드 밸런싱을 통해 요청을 처리하는 경우, 각 서버가 동일한 리소스에 대해 동일한 ETag를 생성하지 않으면 캐시 검증이 제대로 작동하지 않을 수 있다. 이를 해결하기 위해서는 서버 간의 ETag 생성 로직을 일관되게 유지해야 한다.

## 참고
https://tecoble.techcourse.co.kr/post/2020-09-30-ETag-with-Spring/
