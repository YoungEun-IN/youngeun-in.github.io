# RestClientException


![image](https://user-images.githubusercontent.com/46465928/195990705-a1ff6cf5-d9cd-4655-b4ef-c3f849c8362b.png)

## NestedRuntimeException
- RuntimeException의 root cause를 다루기 쉽게 래핑한 예외 클래스
- 내부적으로는 NestedExceptionUtils 라는 유틸리티 클래스를 이용한다.

### RestClientException
- 클라이언트 사이드의 HTTP 에러를 만났을 때 던져지는 기본 예외 클래스

#### RestClientResponseException
- 실제 HTTP 응답 데이터를 포함하고 있는 예외클래스들의 공통 기반 클래스
- int 타입의 rawStatusCode를 가지고 있다.
  - int rawStatusCode
  - String statusText
  - byte[] responseBody: getResponseBodyAsString() 메서드로 읽어올 수 있다.
  - HttpHeaders responseHeaders
  - String responseCharset

##### HttpStatusCodeException
- HttpStatus (enum)를 기반으로 하여 만든 추상 클래스
- getStatusCode() 메서드를 통해 HttpStatus를 읽어올 수 있다.
- 직접쓰기 보다는 상속받은 아래 두 클래스를 사용한다.
  - HttpClientErrorException : 4xx 대 응답을 받았을 때 던져지는 예외
  - HttpServerErrorException: 5xx 대 응답을 받았을 때 던져지는 예외

##### UnknownHttpStatusCodeException
- HTTP 응답 코드가 허용 범위를 넘어섰을 경우 던져진다.
- 499 (Nginx) ...

#### ResourceAccessException
- I/O를 하는 도중 에러가 발생했을 때 던져진다.

## 참고
https://namocom.tistory.com/712

