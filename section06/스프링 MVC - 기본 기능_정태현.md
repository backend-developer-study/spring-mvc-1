# 06_스프링 MVC - 기본 기능

## 로깅 간단히 알아보기

### 어노테이션 정리

- `@RestController`

  - `@Controller` 어노테이션은 반환 값이 `String`이면 뷰 이름으로 인식해서 뷰를 찾고 해당 뷰를 렌더링 시킨다.
  - `@RestController` 어노테이션은 반환 값으로 뷰를 찾는 것이 아니라, **HTTP 메시지 바디에 바로 입력한다.**
  - 따라서 실행 결과로 문자열을 받을 수 있다. 이 부분은 `@ResponseBody`와 관련이 있는데 강의 후반 섹션 알아보자.

- `@Slf4j`

  - Lombok 의존성을 추가하면 사용할 수 있는 어노테이션으로 다음 코드를 자동으로 생성해서 로그를 선언해준다.

  - ```java
    private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(RequestHeaderController.class);
    ```

### 로그

- 로그를 확인하면 다음과 같은 포맷을 갖는 것을 알 수 있다.
  - 시간, 로그 레벨, 프로세스 ID, 쓰레드 명, 클래스 명, 로그 메시지
- 로그 레벨
  - (레벨이 높다) TRACE > DEBUG > INFO > WARN > ERROR (레벨이 낮다)
  - 개발 서버는 `debug`레벨의 로그를 사용
  - 운영 서버는 `info`레벨의 로그를 사용
- 로그 레벨 설정
  - `application.properties` 또는 `application.yml`에서 설정 가능
- 올바른 로그 사용법
  - `log.debug("data="+data)` [❌]
  - `log.debug("data={}", data)` [⭕]
- 로그 사용시 장점
  - 쓰레드 정보, 클래스 이름과 같으 부가 정보를 함께 볼 수 있고, 출력 모양을 변경할 수 있다.
  - 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영 서버에서는 출력하지 않는 등 로그 레벨을 상황에 맞게 변경할 수 있다.
  - 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등, 로그를 별도의 위치에 남길 수 있다.
    - 특히, 파일로 남길 때는 일별, 특정 용량에 따라 로그를 분할하는 것도 가능하다.
  - 성능도 일반 `System.out`보다 좋다. (내부 버퍼링, 멀티 쓰레드 등등) 실무에서는 꼭 로그를 사용해서 출력하자.

## 요청 매핑

#### PathVariable(경로 변수) 사용

```java
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("userId") String data) {
    log.info("mappingPath userId={}", data);
    return "ok";
}
```

- 최근 HTTP API는 리소스 경로에 식별자를 넣는 스타일을 선호한다.
- `@RequestMapping`은 URL 경로를 템플릿화 할 수 있는데, `@PathVariable`을 사용하면 매칭 되는 부분을 편리하게 조회할 수 있다.
- `@PathVariable`의 이름과 파라미터 이름이 같으면 생략할 수 있다.

### 미디어 타입 조건 매핑 - HTTP 요청 Content-Type, consume

```java
@PostMapping(value = "/mapping-consume", consumes = "application/json")
public String mappingConsumes() {
    log.info("mappingConsumes");
    return "ok";
}
```

- HTTP 요청의 Content-Type 헤더를 기반한 미디어 타입으로 매핑한다.
- 위의 코드를 매핑하기 위해서는 `Content-Type`이 `application/json`이어야 한다.

### 미디어 타입 조건 매핑 - HTTP 요청 Accept, produce

```java
@PostMapping(value = "/mapping-produce", produces = "text/html")
public String mappingProduces() {
    log.info("mappingProduces");
    return "ok";
}
```

- HTTP 요청의 Accept 헤더를 기반으로 미디어 타입으로 매핑한다.
- 위의 코드를 매핑하기 위해서는 `Accept`가 `text/html`이어야 한다.

## HTTP 요청 - 기본, 헤더 조회

```java
@Slf4j
@RestController
public class RequestHeaderController {
    @RequestMapping("/headers")
    public String headers(
            HttpServletRequest request,
            HttpServletResponse response,
            HttpMethod httpMethod,
            Locale locale,
            @RequestHeader MultiValueMap<String, String> headerMap,
            @RequestHeader("host") String host,
            @CookieValue(value = "myCookie", required = false) String cookie) {
        log.info("request = {}", request);
        log.info("response = {}", response);
        log.info("httpMethod = {}", httpMethod);
        log.info("locale = {}", locale);
        log.info("headerMap = {}", headerMap);
        log.info("header host = {}", host);
        log.info("myCookie = {}", cookie);

        return "ok";
    }
}
```

- `@RequestHeader MultiValueMap<String, String> headerMap`
  - 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다
- `@RequestHeader("host") String host`
  - 특정 HTTP 헤더를 조회한다. 
  - 속성 
    - 필수 값 여부: `required`
    - 기본 값 속성: `defaultValue`
- `@CookieValue(value = "myCookie", required = false) String cookie`
  - 특정 쿠키를 조회한다.
  - 속성
    - 필수 값 여부: `required`
    - 기본 값 속성: `defaultValue`

- 참고) `MultiValueMap`

  - Map과 유사하지만, 하나의 키에 여러 값을 받을 수 있다.

  - HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용한다.

    - ex) keyA=value1&keyA=value2

  - ```java
    MultiValueMap<String, String> map = new LinkedMultiValueMap();
    map.add("keyA", "value1");
    map.add("keyA", "value2");
    ```

## HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form

### HTTP 요청 데이터 조회 - 개요

- HTTP 요청 메시지를 통해 클라리언트에서 서버로 데이터를 전달하는 방법
  1. GET - 쿼리 파라미터
     - 메시지 바디없이, URL에 쿼리 파라미터에 데이터를 포함해서 전달
     - ex) 검색, 필터, 페이징 등에서 많이 사용하는 방식
  2. POST - HTML Form
     - `content-type: application/x-www-form-urlencoded`
     - 메시지 바디에 쿼리 파리미터 형식으로 전달 (username=hello&age=20)
     - ex) 회원 가입, 상품 주문, HTML Form 사용
  3. HTTP message body에 데이터를 직접 담아서 요청
     - HTTP API에 주로 사용
     - 데이터 형식을 주로 JSON을 사용
     - POST, PUT, PATCH

### 요청 파라미터 - 쿼리 파라미터, HTML Form

- `HttpServletRequest`의 `request.getParameter()`를 사용하면 다음 두 가지 요청 파라미터를 조회할 수 있다.
- 아래의 두 전송방식은 형식이 같기 때문에 구분없이 조회할 수 있다.
- 이것을 간단히 `요청 파라미터(request parameter) 조회`라고 한다.
  1. GET - 쿼리 파라미터
  2. POST - HTML Form

```java
@Slf4j
@Controller
public class RequestParamController {
    @RequestMapping("/request-param-v1")
    public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));
        log.info("username = {}, age = {}", username, age);
        response.getWriter().write("ok");
    }
}
```

## HTTP 요청 파라미터 - @RequestParam

```java
@ResponseBody
@RequestMapping("/request-param-v2")
public String requestParamV2(
    @RequestParam("username") String memberName,
    @RequestParam("age") int memberAge
) {
    log.info("username = {}, age = {}", memberName, memberAge);
    return "ok";
}
```

- `@RequestParam` : 파라미터 이름으로 바인딩 
- `@ResponseBody` : View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력

## HTTP 요청 파라미터 - @ModelAttribute

> 만약에 클라이언트로부터 입력 받아야하는 파라미터가 많다면 어떻게 해야할까?

```java
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData) {
    log.info("username = {}, age = {}", helloData.getUsername(), helloData.getAge());
    return "ok";
}
```

- 스프링 MVC는 `@ModelAttribute`이 있으면 다음을 실행한다.
  - `HelloData` 객체를 생성한다.
  - 요청 파라미터의 이름으로 `HelloData` 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력(바인딩)한다.
  - ex) 파라미터 이름이 username 이면 setUsername() 메서드를 찾아서 호출하면서 값을 입력한다.

```java
@ResponseBody
@RequestMapping("/model-attribute-v2")
public String modelAttributeV1(HelloData helloData) {
    log.info("username = {}, age = {}", helloData.getUsername(), helloData.getAge());
    return "ok";
}
```

- `@ModelAttribute`는 생략할 수 있다.
- 그런데 `@RequestParam` 도 생략할 수 있으니 혼란이 발생할 수 있다
- 스프링은 해당 어노테이션을 생략하면 다음과 같은 규칙을 적용한다.
  - `String`, `int`, `Integer` 같은 단순 타입 => `@RequestParam`
  - 나머지 => `@ModelAttribute` (argument resolver 로 지정해둔 타입 외)