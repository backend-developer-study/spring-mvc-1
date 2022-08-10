
# 스프링의 강의 리뷰📽
> LoadMap Part : 스프링 MVC 패턴 1편     
> Section : 06.스프링 MVC - 기본기능    
> CreateDate : 2022.07.29   
> UpdateDate : 2022.07.

### 목차
 - HTTP 요청 메시지
   - [단순 텍스트](#Text) 
   - [JSON](#Json)
 - HTTP 응답
   - [정적리소스, 뷰 템플릿](#StaticResourcesViewTemplate)
   - [HTTP API, 메시지 바디에 직접입력](#HTTPAPI)
 - [HTTP 메시지 컨버터](#MessageConverter)
 - [요청 매핑 핸들러 어댑터 구조](#requsetMappingHandlerAdapter)
<br></br>

### IntelliJ 단축키

<br></br>
<br></br>

# HTTP 요청 메시지
 - HTTP message Body에 데이터를 직접 담아서 요청하는 방식인데, 
   - HTTP API에서 사용한다.(주로 JSON를 사용함.)
## 단순 텍스트<a name="Text"></a>
 - Body에 있는 정보를 받는 방법이 여러가지 있는데, 최종적으로는 마지막 버전을 주로 쓴다.
 - 첫번째 부터 마지막까지 변천과정이라 보면 된다.
   - @PostMapping() 은  생략하였다.
### 첫번째 방법(servlet.inputStream)
  - `HttpServletRequest`에서 InputStream를 사용하여, HTTP 메시지 바디에 접근할 수 있다.
  - `Stream`의 값은 바이트 코드로 되어 있기 때문에, 인코딩 작업이 필요로하다.
  - 반환값도 `HttpServletResponse` 에 있는 Writer를 가져와 입력해주면된다.
```java
public void  requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException{
        ServletInputStream inputStream=request.getInputStream();
        String messageBody=StreamUtils.copyToString(inputStream,StandardCharsets.UTF_8);
        response.getWriter().write("ok");
}
```

### 두번째 방법(InputStream, Writer)
 - `HTTPServlet`요청값과 응답값을 다 사용하는 것이 아니라면, 굳이 다 인자로 가져올 필요가 없다. 
 - 요청서블릿의 InputStream를 직접 받을 수 있고,응답서블릿의 Writer를 직접 받을 수 있다.
```java
 public void  requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
        responseWriter.write("ok");
}
```

### 세번째 방법(HttpEntity)
 - HTTP의 헤더와 바디의 정보를 포함하고 있는 `HttpEntity`로 받아 올 수 있다.
   - view 조회는 안된다.
<p align="center"><img src="https://user-images.githubusercontent.com/104331549/181686251-9a1d6a46-878e-4fb1-b242-6db0940e6856.png"></p>

 - 위그림에서도 알 수 있듯이, getBody(), getHeader()를 통해 바디와 헤더에 접근할 수 있다.
   - `hasBody()`로 바디의 존재유무도 확인할 수 있다.
 - 또한, HttpEntity 생성자로는 크게 4가지 방식으로 만들어 줄수 있는데, 
   - MutiValueMap<> 타입이 아닌 값을 인자로 생성해주면, Body를 만든 HTTP 엔티티가 만들어 진다.
    
<p align="center"><img src="https://user-images.githubusercontent.com/104331549/181686834-3b330adb-46b2-4ac1-8092-8a7c53d37953.png" width="70%"></p>

```java
public HttpEntity<String>  requestBodyStringV3(HttpEntity<String> httpEntity) throws IOException {
        String messageBody = httpEntity.getBody();
        return  new HttpEntity("ok");
        }
```

### 네번째 방법(RequestEntity, ResponseEntity)
- `HttpEntity`를 상속받는 `RequestEntity`와 `ResponseEntity`를 사용하는 방법이다.
- RequestEntity
    <p align="center"><img src="https://user-images.githubusercontent.com/104331549/181693675-8dc852d0-3a74-49d3-bf41-603ea3be2369.png" width="70%"></p>
    <p align="center"><img src="https://user-images.githubusercontent.com/104331549/181693973-59f238a7-9df1-4084-8674-864b4dc3fa4d.png" width="70%"></p>

   - 실제 초기화 하는 방법
    ```java
     RequestEntity
          .post(&quot;https://example.com/{foo}&quot;, &quot;bar&quot;)
          .accept(MediaType.APPLICATION_JSON)
          .body(body)
    ```
  
- ResponseEntity
     <p align="center"><img src="https://user-images.githubusercontent.com/104331549/181688152-5f27084b-162e-4f3b-a2bd-a83a27d7d05e.png" width="70%"></p>  

    - 실제 초기화 방법
    
    ```java
   return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED)
  ```
- 위 예시들을 ` package org.springframework.http;`안에 다 들어 있다.

```java
    @PostMapping("/request-body-string-v4")
    public HttpEntity<String>  requestBodyStringV4(RequestEntity<String> requestEntity) throws IOException {
        String messageBody = requestEntity.getBody();
        return new ResponseEntity<String>( HttpStatus.OK);
    }
```

### 다섯번째방법(어노테이션)
 - @RequestBody : HTTP 메시지 바디 정보를 편리하게 조회할 수 있다
 - @ResponseBody : 응답 결과를 HTTP 메시지 바디에 직접 담아서 전달할 수 있다
   - view를 사용하지 않는다.
```java
@ResponseBody
public String  requestBodyStringV5(@RequestBody String messageBody) throws IOException {
    return "ok";
}
```

<br></br>
<br></br>
## HTTP 요청 메시지 - JSON<a name="Json"></a>
- Body에 있는 Json정보를 받는 방법도 Text를 받아온 방법과 원리 자체는 크게 다르지 않다. 
- 다른 점이 있다면, 매핑작업을 해줘야 한다는 것이다. 
- `@PostMapping`은 생략하였다.

### 첫번째방법(request.getInputStream)
 - 문자로 된 Json값을  Jackson 라이브러리인 objectMapper 를 사용해서 자바 객체(HelloData.class)로 변환
```java
private ObjectMapper objectMapper = new ObjectMapper();
public void requsetBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
    ServletInputStream inputStream = request.getInputStream();
    String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
    HelloData data = objectMapper.readValue(messageBody, HelloData.class);
    response.getWriter().write("ok");
}
```

### 두번째 방법(@RequestBody 타입)
- 바로 배웠던 어노테이션을 적용해보자
- `@RequsetBody`어노테이션을 사용하면, 요청값의 바디를 바로 받을 수 있는데, 
  - 이때 **HTTP 메시지 컨버터**가 내가 원하는 객체타입(혹은 문자)으로 매핑을 해줄 수 있다. 
- 단, 이 방식은 `@RequsetBody`를 생략하면 안된다.
- 또한, `@ResponseBody`를 사용하면, 객체로 반환하여도 응답이 가능하다. 

#### 정리
- @RequestBody 요청 
  - JSON 요청 -> HTTP 메시지 -> 컨버터 객체
- @ResponseBody 응답
  - 객체 HTTP -> 메시지 컨버터 -> JSON 응답

```java
@ResponseBody
public HelloData requestBodyJsonV2(@RequestBody HelloData data) throws IOException {
    log.info("username={}, age={}",data.getUsername(), data.getAge());
    return data;
}
```

### 세번째 방법(HttpEntity<>)
- HttpEntity<T>에서 T에 내가 만든 객체나, 타입을 넣어주면 가능하다.
```java
@ResponseBody
public String requestBodyJsonV4(HttpEntity<HelloData> data) throws IOException {
    HelloData data = httpEntity.getBody();
    log.info("username={}, age={}",data.getUsername(), data.getAge());
    return "ok";
}
```

<br></br>
<br></br>
# HTTP 응답
> 응답 데이터를 만드는 방법도 크게 3가지가 있다.
- 정적 리소스 
  - HTML, css, js 등과 같은 웹브라우저의 정적리소스
- 뷰템플릿 사용
  - 동적인 HTML을 제공할 뷰 템플릿
- HTTP 메시지 사용
  - HTTP API(Json)


## 정적 리소스<a name="StaticResourcesViewTemplate"></a>
- 스프링 부트는 클래스패스의 다음 디렉토리에 있는 정적리소스를 제공한다.
- `src/main/resources`리소스를 보관하는 곳으로,하위에서 아래와 같은 디렉토리를 만들어 제공한다.
  - `/static` , `/public` , `/basic` 
  - 또 클래스패스(classpath)의 시작 경로이다
- 예시
  - 파일 경로 :`src/main/resources/static/basic/hello-form.html `
  - 웹브라우저 URI : `http://localhost:8080/basic/hello-form.html`
<br></br>

## 뷰템플릿
 - 뷰 템플릿을 거쳐서 HTML이 생성되고, 뷰가 응답을 만들어서 전달한다.
 - 일반적으로 HTML을 동적으로 생성하는 용도지만, 뷰 템플릿이 만들 수 있는 다른 것도 가능하다. 

### 뷰 템플릿 기본 경로
`src/main/resources/templates`

## 뷰 템플릿 응답 반환
 - 크게 3가지 방법이 있지만, 마지막 방법은 잘 사용하기 않는다. 


### 첫번째 방법(ModelAndView반환)
- 말그대로, 모델뷰를 직접 만들어 반환시켜주는 방법이다.
- ModelAndView 타입은 소유하고 있는 값이, view, model, HttpStatus등이 있어, 그대로 반환해주면 된다.
```java
@RequestMapping("/response-view-v1")
public ModelAndView responseViewV1(){
    ModelAndView mav =new ModelAndView("response/hello").addObject("data", "hello!");
    return mav;
}
```
### 두번째 방법(String 반환)
- 해당 문자열이 뷰템플릿이 있는 **파일경로**로 되어 있어 반환하면, 해당 경로 뷰 템플릿 파일을 찾아 반환한다.
- 여기서 Model에 담긴 데이터들은 스프링이 뷰파일로 전달한다.
    - [SpringFramework 동작순서 자세한 내용](#SpringFrameworkFlow)
- spring이 뷰 템플릿이 렌더링 되는 곳을 `templates`폴더 하위로 설정이 되어 있다. 
  - 그 이유는 `Thymeleaf`를 라이브러리에 추가하게 되면, 애플리케이션 설정(`application.properties`)에 기본 설정값이 추가된다. 
    ```properties
    spring.thymeleaf.prefix=classpath:/templates/
    spring.thymeleaf.suffix=.html
    ```
  - 기본값이라, 생략가능하고 변경이 필요할때만 설정하면 된다.
  - [Templating Properties 참고링크](https://docs.spring.io/spring-boot/docs/2.4.3/reference/html/appendix-application-properties.html#common-application-properties-templating)
```java
@RequestMapping("/response-view-v2")
public String responseViewV2(Model model){
    model.addAttribute("data", "hello!");
    return "response/hello";
}
```

### 세번째 방법(Mapping()경로로 반환)
- 코드가 명시성이 떨어지고 실용성도 좋지않아 권장되지 않는 방법이다.
```java
@RequestMapping("/response/hello")
public void responseViewV3(Model model) {
    model.addAttribute("data", "hello!!");
}
```

<br></br>
<br></br>

# HTTP 응답 - HTTP API, 메시지 바디에 직접 입력<a name="HTTPAPI"></a>
 - 이미 위에서 한번 다뤘던 내용임으로 중복되는 내용은 넘어간다.

### @ResponseStatus()
 - 아래처럼 반환값이 내가만든 클래스여서, HTTP상태코드를 반환하지 못하는 경우에는,`@ResponseStatus` 애노테이션을 사용하여, 원하는 HTTP상태코드를 반환시킬 수 있다.
 - 물론 애노테이션이기 때문에 응답 코드를 동적으로 변경할 수는 없다.
```java
 @ResponseStatus(HttpStatus.OK)
 @ResponseBody
 @GetMapping("/response-body-json-v2")
 public HelloData responseBodyJsonV2() {
     HelloData helloData = new HelloData();
     helloData.setUsername("userA");
     helloData.setAge(20);
     return helloData;
}
```

<br></br>
<br></br>

# HTTP 메시지 컨버터<a name="MessageConverter"></a>
 - HTTP 요청을 모델에 바인딩하고 클라이언트에 보낼 HTTP 응답을 만들기 위해 뷰를 사용했던 방식과는 달리,  
   HTTP 메시지 컨버터는 HTTP 요청 본문과 HTTP 응답 본문을 통째로 메세지로 다루는 방식이다.
 - 주로 `XML`이나 `JSON`을 이용한 AJAX기능이나 웹 서비스를 개발할 때 사용된다.
 - 스프링의 `@RequestBody`와 `@ResponseBody`를 통해 HTTP 메시지 컨버터를 자동 사용할 수 있다.
   - 혹은 HttpEntity(RequestEntity), HttpEntity(ResponseEntity)에서도 HTTP 메시지 컨버터가 적용된다.
   - 요청에 Body가 없는 경우에는 `@RequestBody`를 사용할 수 없고, 이 경우엔 `@RequestParam`이나 `@ModelAttribute`를 사용해야 한다.

<p align="center"><img src="https://user-images.githubusercontent.com/104331549/181908686-3bc0e998-8891-4642-b28b-c4c98caaa17a.png" width="70%"></p>


## 메시지 컨버터 자동생성
- 이렇게 사용되는 메시지 컨버터는 Springframeworboot를 사용하면 자동 생성된다.
  - `BackgroundPreinitializer`클래스는 이름그대로 미리초기화되는걸 관리하는 클래스인데, 내부적으로 MessageConverter를 생성하여 실행 시키는 것을 알 수 있다.

<p align="center"><img src="https://user-images.githubusercontent.com/104331549/181909826-1c86da4d-67ec-4c43-b552-8fff75e410b4.png" width="70%"></p>

- 그리고 `AllEncompassingFormHttpMessageConverter()` HttpMessageConverter를 생성해주는 이 생성자를 따라 들어 가면
  - xml과 Json관련된  HttpMessage들을 Mapping 시켜주는 컨버터가 생성된다는 것을 알 수 있다.
  - `Json`을 `model`로 매핑해주는 `MappingJackson2HttpMessageConverter`를 만들어 주는게 보인다.
<p align="center"><img src="https://user-images.githubusercontent.com/104331549/181913762-e098a307-d5c2-4d57-a5f6-c2603961d915.png" width="70%"></p>

> 그럼 나머지 메시지컨버터는 어딨을까?
- 위  `AllEncompassingFormHttpMessageConverter`를 다시 보면, `FormHttpMessageConverter`를 상속하고 있는 것을 알 수 있다. 
- 그래서 `FormHttpMessageConverter`를 확인해보면 HttpMessageConverter를 List로 가지고 있다. 
- 우선순위는 List에 추가된 순서대로 탐색을 하기에, 추가된 순서를 따른다.
  - ByteArrayHttpMessageConverter
  - StringHttpMessageConverter
  - SourceHttpMessageConvreter

<p align="center"><img src="https://user-images.githubusercontent.com/104331549/181914366-a587e903-2dba-4058-b4c8-483ba703201c.png" width="70%"></p>

### 자동생성된 메시지 컨버터 종류
1. ByteArrayHttpMessageConverter : byte[] 데이터를 처리한다
    - 미디어타입은 모든 것을 다 지원한다.
    - 즉,**요청** 파라미터에 `@RequestBody` `byte[] param`과 같이 작성하면 모든 요청을 다 byte배열로 받을 수 있다는 말이다.
    - **응답** 리턴타입을 `byte[]`로 했을 경우 `Content-Type`이 `applcation/octet-stream`으로 설정되어 전달된다.
    - 파일 및 이미지 다운로드/업로드일때 쓰인다고 한다.
2. StringHttpMessageConverter : String 문자로 데이터를 처리한다.
   - 미디어타입은 모든 것을 다 지원한다.
   - **요청** 파라미터에 사용할 경우 `HTTP 본문을 그대로 String`으로 가져올 수 있다.
   - **응답** 리턴에 사용할 경우 단순 문자열을 그대로 전달해줄 수 있다. `Content-Type`은 `text/plain`
3. SourceHttpMessageConvreter
    - 미디어타입은 모든 것을 다 지원한다.
    - `Content-Type`이 `application/octet-stream`를 지원한다. 알려지지않은 파일 타입은 이타입을 사용한다.
    - 브라우저들은 이런 파일들을 다룰 때, 사용자를 위험한 동작으로부터 보호하도록 개별적인 주의를 기울여야 한다.
4. FormHttpMessageConverter
   - 보이다시피, 지원하는 미디어 타입은 `MediaType.APPLICATION_FORM_URLENCODED`, `MediaType.MULTIPART_FORM_DATA`,`MediaType.MULTIPART_MIXED`이다.
   - 즉, 정의된 폼 데이터를 주고받을 때 사용할 수 있다
5. MappingJackson2HttpMessageConverter
   - 미디어타입은 `MediaType.APPLICATION_JSON`만 지원한다.
   - Jackson의 objectMapper를 이용하여 JSon를 변환한다.
   - 변환하는 오브젝트 타입의 제한은 없지만, **프로퍼티를 가진 자바빈 스타일(커스텀 객체)**이나 **HashMap**을 이용해야 정확한 변환 결과를 얻을 수 있다.

<br></br>

## @EnableWebMvc
- 이렇게 자동생성되는 메시지컨버터들은, 애노테이션 `@EnableWebMvc`안에 있는 `WebMvcConfigurationSupport`에 의해 재정의되고,더 많은 메시지컨버터들이 등록되어 관리받게 된다.
- 애초에 `@EnableWebMvc`은 `WebMvcConfigurer인터페이스`를 상속받는 구현체 `커스텀Configuration`에 붙이는 애노테이션이다.
  - `@EnableWebMvc` MVC 관련된 bean들을 수정할때 쓰인다고 한다.
<p align="center"><img src="https://user-images.githubusercontent.com/104331549/181915711-cc8ce67d-4594-4ca0-8e0f-b94b96d752a5.png" width="70%"></p>

<br></br>
<br></br>

# 요청 매핑 핸들러 어댑터 구조<a name="requsetMappingHandlerAdapter"></a>
> 내가 알아보고자했던 RequestMappingHandlerAdapter에 대해 나온다.
- 위에서 다뤘던 메시지 컨버터는 어디서 사용되는 것일까?
- 또, model은 어떻게 view(혹은 뷰템플릿)으로 넘어 가는 것일까?

> 답은 @RequestMapping 을 처리하는 핸들러 어댑터인 RequestMappingHandlerAdapter (요청 매핑 헨들러 어뎁터)에 있다.

## RequestMappingHandlerAdapter
 - Dispatcher서블릿과 컨트롤러 사이에 있는 핸들러 어댑터로, `@RequsetMapping`에 관련된 요청들은 RequestMappingHandlerAdapter를 지난다.
<p align="center"><img src="https://user-images.githubusercontent.com/104331549/181917538-1b14b3fc-cc73-4fb0-97fc-2b98cf34c0f9.png" width="70%"></p>

 - 이 핸들러를 보다 자세히 보면 동작방식은 아래와 같다.
<p align="center"><img src="https://user-images.githubusercontent.com/104331549/181917577-30e4b4e8-21ca-4e2f-bf4f-431ed35d139a.png" width="70%"></p>

### ArgumentResolver
 - 애노테이션 기반의 컨트롤러는 매우 다양한 파라미터를 사용할 수 있었다.
   - `HttpServletRequest` , `Model`, `@RequestParam` , `@ModelAttribute` , `@RequestBody` , `HttpEntity` 등등
 - 이렇게 파라미터를 유연하게 처리할 수 있는 이유가 바로 `ArgumentResolver` 덕분이다.
   - 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성한다.
 - 스프링은 30개가 넘는 `ArgumentResolver` 를 List<>화 되어 기본으로 제공한다
 - 정확히는 `HandlerMethodArgumentResolver` 인데 줄여서 `ArgumentResolver`라고 부른다/
 - 인터페이스를 사용하여, 내가 원하는 커스텀 `ArgumentResolver`를 만들 수도 있다.

### ReturnValueHandler
 - `HandlerMethodReturnValueHandler` 를 줄여서 `ReturnValueHandler` 라 부른다
 - `ArgumentResolver` 와 비슷한데, 이것은 반대로 응답 값을 변환하고 처리한다.
 - 스프링은 10여개가 넘는 `ReturnValueHandler` 를 List<>화 되어 기본으로 지원한다
   - `ModelAndView` , `@ResponseBody` , `HttpEntity` , `String`
<p align="center"><img src="" width="70%"></p>

## 느낀점 😌

## Spring Framework Controller 동작 알아보기
 > Model 에 담긴 데이터들은 스프링이 뷰파일로 전달하게 된다.  
 > Model에 담는것이 일반적이긴 하나 사용자가 지정한 객체로 전달할 수도 있습니다.
 - 위 2가지가 가능한 이유는 이전 섹션에서 배운 `@ModelAttribute` 어노테이션 덕분이다.
 - 다시 한번 언급하지만, 실제 개발에서는
     1. 요청 파라미터를 받아서
     2. 필요한 객체를 만들고
     3. 그 객체에 값을 넣어줘야한다.
>  스프링은 이 과정을 완전히 자동화해주는 `@ModelAttribute` 기능이 있다.
 - 하지만, 위 과정 말고도 한가지 더 해주는 작업이 있는데, 이것이 자동으로 Model객체가 추가되고 view단으로 전달된다는 것이다.
 - 예를 들어보자
### 예시
 - 해당뷰 `/response-view-v2` url를 매핑하는 메소드는 위치만 반환하는 메소드이다.
 - 그리고 `@ModelAttribute`어노테이션을 가지는 model.addAttribute("data", "hello!")을 해주는 메소드를 만들었다.
```java
@RequestMapping("/response-view-v2")
public String responseViewV2(String data){
        return "response/hello"; 
}

@ModelAttribute
public void addAttributes(Model model) {
        model.addAttribute("data", "hello!");
}
```
<p align="center"><img src="https://user-images.githubusercontent.com/104331549/181719795-51ccd4a0-b91d-49ba-8412-788d31d30327.png" width="70%"></p>

 - 잘 들어간걸 확인할 수 있다. 
 - 일반적으로 Spring MVC는 요청 핸들러 메소드를 호출하기 전에 항상 @ModelAttribute 메소드를 먼저 호출한다.
 - 기본적으로 @ModelAttribute 메서드는 @RequestMapping 으로 주석이 달린 컨트롤러 메서드가 호출되기 전에 호출된다.
   - 컨트롤러 메서드 내에서 처리가 시작되기 전에 **모델 개체를 만들어야 하기 때문**이다.

### 예시2 
 - 이번엔 모델로 사용할 객체를 만들어 보자
 - 뷰템플릿 하나 생성
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <p th:text="${name}">empty</p>
    <p th:text="${id}">empty</p>
</body>
</html>
```
- 객체
```java
@Data
public class Employee {
    private long id;
    private String name;

    public Employee(long id, String name) {
        this.id = id;
        this.name = name;
    }
}
```
- 컨트롤러
```java
@PostMapping("/addEmployee")
    public String submit(@ModelAttribute("employee")Employee employee, Model model){
        model.addAttribute("name", employee.getName());
        model.addAttribute("id", employee.getId());
        return "response/employeeView";
    }
```
<p align="center"><img src="https://user-images.githubusercontent.com/104331549/181721223-faa216a7-5835-4ed9-9393-4a1d85256b30.png" width="70%"></p>

 - 이렇게 Model이 자동으로 뷰에 주입되는 것을 알 수 있다.
 - 게다가`@ModelAttribute` 생략가능하다.

> 아직 무언가 해결이 된 느낌은 아니다.  찾아보니까 `RequestMappingHandlerAdapter` 라는 핸들러에서 model을 넘겨주는 작업을 해주는 것 같은데  
>  추후 알아봐야될것 같다.  

### 참고 링크
- [Spring Framework (스프링프레임워크) 기본 동작 순서 및 구조](https://intro0517.tistory.com/151)
- [@modelattribute](https://www.baeldung.com/spring-mvc-and-the-modelattribute-annotation)
- [@modelattribute](https://hoit1302.tistory.com/64)
- [Spring-web-MVC](https://incheol-jung.gitbook.io/docs/q-and-a/spring/enablewebmvc#spring-mvc-beans)
- [메시지 컨버터의 종류](https://joont92.github.io/spring/MessageConverter/)
- [WebMvcConfigurationSupport](https://kim0lil.github.io/skfactory.github.io/2020/06/03/page7.html)
- [@EnableWebMvc를 사용하는 이유](https://incheol-jung.gitbook.io/docs/q-and-a/spring/enablewebmvc)
- [ArgumentResolver](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments)
- [ReturnValueHandler](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types)
