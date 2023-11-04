# RequestBody, ModelAttribute, RequestParam


## @RequestBody
**클라이언트가 body에 JSON 또는 XML 형태로 값(보통 객체)을 담아 전송하면, body의 내용을 다시 Java Object(객체)로 변환해주는 역할을 수행한다.** 이 데이터는 Spring에서 관리하는 MessageConverter들 중 하나인 Jackson2HttpMessageConverter를 통해 Java 객체로 변환된다.

### DTO에 기본 생성자가 필요한 이유
 RestController에서 `@RequestBody의` 바인딩은 Jackson라이브러리의 ObjectMapper가 해준다. **ObjectMapper는 @RequestBody가 Property로 구현되어 있거나 생성을 위임한 경우가 아니라면 기본 생성자로 생성한다.** 따라서 두 상황이 아니라면 기본 생성자는 꼭 필요하다.

### Setter가 필요없는 이유
ObjectMapper는 Setter 또는 Getter로 DTO의 필드를 가져온다. 그리고 **setter를 사용하지 않는 경우에는 reflection을 사용해서 필드 값들을 넣어준다.**

## @ModelAttribute
클라이언트가 전송하는 HTTP parameter(URL 끝에 추가하는 파라미터), HTTP Body 내용을 1:1로 객체에 데이터를 바인딩한다. **HTTP Body 내용은 multipart/form-data 형태이다.** 따라서 Form 안에 input 값을 담아 보낼 때 유용하게 사용된다. 

### Setter가 필요한 이유
**객체 바인딩 시 별다른 설정이 없다면 Spring 에서는 WebDataBinder의 기본 값 할당 방법인 Java Bean 방식을 사용한다.** 그러므로 Setter가 없으면 작동하지 않는다.

## @RequestParam
**HTTP 요청 parameter를 변수로 1대 1로 Mapping 해준다.** default는 `required = true` 이다. 잘못된 파라미터값이 들어오면 400 BadRequest를 발생시킨다.

## 참고
https://parkadd.tistory.com/70

