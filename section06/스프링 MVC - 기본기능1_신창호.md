
# 스프링의 강의 리뷰📽
> LoadMap Part : 스프링 MVC 패턴 1편     
> Section : 06.스프링 MVC - 기본기능    
> CreateDate : 2022.07.23  
> UpdateDate : 2022.07.28

### 목차
 - [프로젝트 생성](#CreateProject)
 - [로깅 간단히 알아보기](#LoggingTest)
 - [요청 매핑](#RequestMapping)
 - HTTP 요청 
   - [기본, 헤더 조회](#BasicAndHeader)
   - 파라미터 
     - [쿼리 파라미터, HTML Form](#Parameter) 
     - [@RequestParam](#RequestParam)
     - [@ModelAttribute](#ModelAttribute)
<br></br>
### IntelliJ 단축키
<br></br>
<br></br>

# 프로젝트 생성<a name="CreateProject"></a>
- 프로젝트 선택
  - Project: Gradle Project
  - Packaging: **Jar**
  - java : 11
- Dependencies: Spring Web, Thymeleaf, Lombok

### Jar를 선택하는 이유 
 - Jar의 경우
   - 사용하면 항상 내장 서버(톰캣등)을 사용한다. 
   - `webapp` 경로도 사용하지 않는다. 
   - 즉, 내장 서버 사용에 최적화 되어 있는 기능
 - War의 경우 
   - JSP를 사용할 수 있다.
   - 내장 서버도 가능은 하지만, 주로 외부 서버에 배포하는 목적으로 사용

### Welcome 페이지 만들기
 - Jar를 사용한다면,  `/resource/static/index.html` 이 경로에 index.html파일을 만들면 웰컴페이지로 처리해준다.

<br></br>
<br></br>

# 로깅 간단히 알아보기<a name="LoggingTest"></a>
 - 별도의 **로깅 라이브러리**를 사용해서 로그를 출력해보자. 
   - 깊게 들어가면 끝이 없는 영역이다.

### 로깅 라이브러리 
- 스프링 부트 로깅 라이브러리( spring-boot-starter-logging )가 함께 포함된다.
  - SLF4J : 인터페이스 로그 라이브러리
  - Logback : 구현체 로그 라이브러리 
- 로그 라이브러리는 `Logback`, `Log4J`, `Log4J2` 등등 수 많은 라이브러리가 있는데, 그것을 통합해서 인터페이스로 제공하는 것이 바로 `SLF4J` 라이브러리다.
- 실무에서는 스프링부트가 기본으로 제공하는 `Logback`을 대부분 사용한다. 

<br></br>
## 기존 sout과 log 비교하기
### 코드 
```java
@Slf4j
@RestController
public class LongTestController {
//    Slf4j 어노테이션으로 처리가능(lombok라이브러리)
//    private final Logger log = LoggerFactory.getLogger(getClass()); //LongTestController.class
    
    @GetMapping("/log-test")
    public String logTest() {
        String name = "Spring";

        System.out.println("name = " + name);
        log.info(" info log={}", name);

        return "ok";
    }
}
```
### 출력확인 
```shell
name = Spring
2022-07-23 17:44:59.814  INFO 19792 --- [nio-8080-exec-1] h.springmvc.basic.LongTestController     :  info log=Spring
```
 - `System.out.println()`의 경우 ()안에 대한 결과만 찍힌다.
 - 하지만 log.info()의 경우 다양한 정보가 찍힌다.

<p align="center"><img src="https://user-images.githubusercontent.com/104331549/180598373-f9e1fca1-d03c-4baf-9778-296665268c85.png" width="80%"></p>

<br></br>

### 꿀팁
> @RestController 와 @Controller의 차이 
 - 이 둘의 가장 큰 차이는 반환값에 있다.
   - @Controller의 경우 반환값이 응답할 view를 반환한다. 
     - view가 아닌 Data를 반환할 수는 있으나, @ResponseBody 어노테이션을 활용해주어야 한다. 
   - @RestController의 경우 반환값이 응답할 API를 반환한다.
     - 주 용도는 Json 형태로 객체 데이터를 반환하는 것

<br></br>

### 로그 레벨
 - LEVEL: `TRACE` > `DEBUG` > `INFO` > `WARN` > `ERROR`
 - 보통 아래 로그 문구 5가지를 같이 실행시켜도, `INFO` , `WARN` , `ERROR` 만 실행된다.
 - 개발 서버는 debug 출력
   - 운영 서버는 info 출력
```java
log.trace("trace log={}", name);
log.debug("debug log={}", name);
log.info("info log={}", name);
log.warn("warn log={}", name);
log.error("error log={}", name)
```
- 추가적으로 Error 위에, `FATAL` 이라고 하나 더 있다.
  - [참고링크](https://stackoverflow.com/questions/2031163/when-to-use-the-different-log-levels)
- 개발 실무에서 생각보다 `trace` 로그는 잘 사용하지않는다는데, 알고리즘이 있는 경우 `trace`는 각 단계에 대한 정보 최상의 수준으로 볼수 있다.
- 일반적으로 `trace`로그는 `debug`로그도 포함한다.
  - (`debug`에 모든 `warn` 및 `error`가 포함되는 것처럼).
  - 그래서 너무 출력되는 로그가 방대하다보니 성능을 크게 저하시킨다고 한다.
  - [참고링크](https://softwareengineering.stackexchange.com/questions/279690/why-does-the-trace-level-exist-and-when-should-i-use-it-rather-than-debug)
> 만약 debug나 trace를 띄우고 싶다면, 로그레벨을 설정해줘야한다.

<br></br>

### 로그 레벨 설정
 - `application.properties`
```properties
#전체 로그 레벨 설정(기본 info)
logging.level.root=info
#hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=debug 
```
 - 기본값은 `info`로 돼있고, `level`뒤에 `hello.springmvc`를 붙여 해당 패키지와 하위 로그 레벨까지 설정할 수 있다. 
 - 설정값이 `trace`시, 전부 출력, `debug`시 `trace`만 빼고 출력된다.

<br></br>

### 로그 사용시 장점
- 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고, 출력 모양을 조정할 수 있다.
- 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영서버에서는 출력하지 않는 등 **로그를 상황에 맞게 조절할 수 있다**
- 시스템 아웃 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등, 로그를 별도의 위치에 남길 수 있다.
  - 특히 파일로 남길 때는 일별, 특정 용량에 따라 로그를 분할하는 것도 가능하다.
  - 즉, **내가 원하는 곳(필요한 곳)에 로그를 남겨놓을 수 있게된다.**
- 성능도 일반 `System.out`보다 좋다

<br></br>
<br></br>

# 요청 매핑<a name="RequestMapping"></a>

## @RequestMapping()
### HTTP 메서드
  - 다시한번 더 언급하자면 `@RequestMapping()`을 사용하게 되면, 어떠한 HTTP 메서드를 사용하더라도 무관하게 호출된다. 
  - GET 요청만 매핑하려면 method를 별도로 입력해 줘야한다.`value= "url경로" ,method = RequestMethod.GET`
    - POST 요청만 매핑하려면 `method = RequestMethod.POST` 추가 
    - PATCH, DELETE, PUT 등도 동일하다.
### 다중 매핑 
 - 대부분 어노테이션 속성을 배열로 제공하기 때문에 `@RequestMapping({"/hello-basic", "/hello-go"})` URL 다중 매핑이 가능하다.
### 둘다 허용
   - 원래 URL상의 주소값 `/hello-basic` 과 `/hello-basic/`은 다른 URL로 인식하지만, Spring에서 매핑할때는 둘 다 같은 URL요청으로 인식하여 매핑한다.
     
<br></br>

## PathVariable(경로 변수)

### 사용법
 - 최근 HTTP API는 다음과 같이 리소스 경로에 **식별자**를 넣는 스타일
 - `@RequsetMapping`은 URL경로를 템플릿화(식별자넣기) 할 수 있는데, `@PathVariable`로 조회할 수 있다.
```java
@GetMapping("mapping/{userId}")
public String mappingPath(@PathVariable("userId") String data){
    log.info("mappingPath userId={}", data);
    return "ok";
}
```
 - `@PathVariable` 의 이름과 파라미터 이름이 같으면 생략할 수 있다.
```java
@GetMapping("mapping/{userId}")
public String mappingPath(@PathVariable String userId){ // 변수명을 맞추면, @PathVariable안에 속성 생략 가능
    log.info("mappingPath userId={}", userId); 
    return "ok";
}
```

### 다중사용
 - 식별자를 다중으로 넣을 수도 있다. 
```java
@GetMapping("mapping/users/{userId}/orders/{orderId}")
public String mappingPath(@PathVariable String userId, @PathVariable Long orderId){
    log.info("mappingPath userId={}, orderId={}", userId, orderId);
    return "ok";
}
```

### 특정 파라미터 조건 매핑
- HTTP 요청 URL의 parameter값에 특정 파라미터가 있거나 없는 조건을 추가할 수 있다.
  - 잘 사용하지는 않음
  - 아래는 mode =debug라는 값이 있어야한 HTTP요청이 성공한다.
```java
@GetMapping(value = "/mapping-param", params = "mode=debug")
```

### 특정 헤더 조건 매핑
 - 파라미터 매핑과 비슷하지만, HTTP 헤더를 사용한다
 - Postman으로 테스트

<br></br>
## 미디어 타입 조건 매핑 
### HTTP 요청 Content-Type [consumes]
 - HTTP 요청의 Content-Type헤더를 기반으로 미디어 타입으로 매핑한다.
 - 만약, 맞지 않으면 HTTP 415 상태코드`Unsupported Media Type`을 반환
```java
@PostMapping(value = "/mapping-consume", consumes = MediaType.APPLICATION_JSON_VALUE) // "application/json"
public String mappingConsumes(){
    log.info("mappingConsumes");
    return "ok";
}
```
- json타입 말고도 다른 타입도 가능하다. 
```java
consumes = "text/plain"
consumes = {"text/plain", "application/*"}
consumes = MediaType.TEXT_PLAIN_VALUE
```

### HTTP 요청 Accept [produce]
 - Accept 헤더를 기반으로 미디어 타입으로 매핑한다.
 - 만약 맞지 않으면 HTTP 406 상태코드(Not Acceptable)을 반환한다.

```java
@PostMapping(value = "/mapping-produce", produces = MediaType.TEXT_HTML_VALUE) //"text/html"
public String mappingProduces() {
    log.info("mappingProduces");
    return "ok";
}
```
- text/html 타입말고 다른 타입으로도 클라이언트가 받을 수 있는 지 확인 가능하다.
```java
produces = "text/plain"
produces = {"text/plain", "application/*"}
produces = MediaType.TEXT_PLAIN_VALUE
produces = "text/plain;charset=UTF-8
```

<br></br>
<br></br>

# HTTP 요청 - 기본, 헤더 조회<a name="BasicAndHeader"></a>
 - HTTP 요청이 들어왔을때 , HTTP 헤더 정보를  원하는데로 조회할 수 있다. 

```java
@RequestMapping("/headers")
public String headers(HttpServletRequest request,
                      HttpServletResponse response,
                      HttpMethod httpMethod,
                      Locale locale,
                      @RequestHeader MultiValueMap<String, String> headerMap,
                      @RequestHeader String host,
                      @CookieValue(value = "myCookie", required = false) String cookie){
    return "ok";
}
```
 - HttpServletRequest
 - HttpServletResponse 
 - HttpMethod : HTTP 메서드를 조회한다.(org.springframework.http.HttpMethod)
 - Locale : Locale 정보를 조회한다.(Ko-KR 1순위)
   - LocaleResolver인터페이스를 사용하여 더 자세히 조회할 수 있음
   - LocaleResolver의 종류  
   <p align="center"><img src="https://user-images.githubusercontent.com/104331549/180949406-d37e5359-d86d-4956-9881-8b999bd4eca5.png" width="80%"></p>
    
   - LocaleResolver를 이용하여 Locale 변경할수 있다.(블로그서비스에 적용됨)

 - @RequestHeader MultiValueMap<String, String> headerMap
   - 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다.
   - MAP과 유사한데, 하나의 키에 여러 값을 받을 수 있다
     - HTTP header 혹은 HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용한다.
     - 저장할때는 키값, 벨류값이지만, 실제 맵안은 키값은 String, value값은 List<String>으로 된 자료구조이다.  

<img src="https://user-images.githubusercontent.com/104331549/181402658-846e6189-de56-4971-a04e-bf5bc97b44e2.png" >

 - @RequestHeader("host") String host
   - 특정 HTTP 헤더를 조회한다.
   - 속성
     - 필수 값 여부: required
     - 기본 값 속성: defaultValue 
       - 해당 이름값을 가진 헤더정보가 없다면 기본값으로 들어간다. 
 - @CookieValue(value = "myCookie", required = false) String cookie
 - 특정 쿠키를 조회한다.
 - 속성
   - 필수 값 여부: required
   - 기본 값: defaultValue

<br></br>
<br></br>

# HTTP 요청 파라미터 
> 클라이언트에서 서버로 요청 데이터를 전달할 때는 주로 다음 3가지 방법
 - GET - 쿼리 파라미터
 - POST - HTML Form
 - HTTP message `body`에 데이터를 직접 담아서 요청

## 쿼리 파라미터, HTML Form<a name="Parameter"></a>
- `request.getParameter()`를 사용하면 두가지 요청(GET-쿼리파라미터, POST-HTML Form) 파라미터를 조회할 수 있다. 
- `HttpServletRequest`에서 제공하는 메소드 방법이다.
```java
@RequestMapping("/request-param-v1")
public void requestParamV1(HttpServletRequest request , HttpServletResponse response) throws IOException{
    String username=request.getParameter("username"); 
    int age=Integer.parseInt(request.getParameter("age"));
    }
}
```

## @RequestParam<a name="RequestParam"></a>
 - 스프링에서 제공하는 `@RequestParam`으로 편리하게 사용가능!
 - `RequestParam`으로 들어오는 키값과 변수명이 동일하면 생략가능하다.
   - 기본적으로 이름이 동일하다면 속성으로 `value`값이 생략가능하며, 
   - 파라미터 타입이 `String` , `int` , `Integer` 등의 단순 타입이면 @RequestParam 도 생략 가능하다.
```java
@ResponseBody
@RequestMapping("/request-param-v3")
public String requestParamV3(
        @RequestParam String username,
        @RequestParam int age){

    log.info("username={}, age ={}",username,age);
    return "ok";
}

```

### 속성값
- value : 키값을 말한다.
- required(기본값 true) : 
  - true 면 필수 값으로 설정된다. (없으면 에러가 난다.)
  - false 면 없어도 되는 값으로 설정된다.(없으면 null값으로 들어간다.)
  - `@RequestParam` 애노테이션 자체가 없으면 `required=false` 를 적용
  - `http://localhost:8080/request-param-required?username=hello` age값이 없어도 아래 코드는 age=null로 실행된다.
```java
@ResponseBody
@RequestMapping("/request-param-required")
public String requestParamV5(String username, Integer age){
        log.info("username={}, age ={}",username,age);
        return "ok";
        }
```

- defaultValue : 기본 값을 적용할 수 있다.
  - 이미 기본 값이 있기 때문에 required 는 의미가 없다.
  - defaultValue 는 **빈 문자의 경우에도** 설정한 기본 값이 **적용**된다.

```java
@ResponseBody
@RequestMapping("/request-param-default")
public String requestParamV6(
    @RequestParam(required = true, defaultValue = "guest") String username,
    @RequestParam(required = false, defaultValue = "-1") int age
    ){
    return "ok";
}
```

### Map<>으로 받기
 - `@RequestParam Map<>`
   - Map(key=value)
 - `@RequestParam MultiValueMap<>`
   - MultiValueMap(key=[value1, value2, ...] 

> 파라미터 값이 1개가 확실하면 Map을 사용해도 되지만, 그게 아니라면 `MultiValueMap` 사용을 권장하자!

<br></br>
<br></br>

## @ModelAttribute<a name="ModelAttribute"></a>
 - 실제 개발에서는 
   1. 요청 파라미터를 받아서
   2. 필요한 객체를 만들고
   3. 그 객체에 값을 넣어줘야한다.
 - 하지만,스프링은 이 과정을 완전히 자동화해주는 `@ModelAttribute` 기능이 있다.

```java
@Data
public class HelloData {
    private String username;
    private int age;
}
```
 - 롬복 `@Data`
   - `@Getter` , `@Setter` , `@ToString`, `@EqualsAndHashCode` , `@RequiredArgsConstructor`를 자동으로 적용해준다
   - `@EqualsAndHashCode`관련 내용을 정리하다가 길어져서 하단에 [별도정리](#EqualsAndHashCode)

```java
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData){
    log.info("username={}, age={}", helloData.getUsername(),helloData.getAge());
    return "ok";
}
```
 - `@ModelAttribute` 생략가능하다. 단, 다음과 같은 규칙을 적용한다.
   - `String` , `int` , `Integer` 같은 단순 타입 빼고 나머지타입(커스텀 객체타입 포함)
   - argument resolver로 지정해둔 타입은 생략할 수 없다.

### 프로퍼티 
- 프로퍼티는 소유물이라는 뜻으로 클래스의 변경가능한 필드를 가르킨다.
- 일부 객체 지향 프로그래밍 언어에서 필드(데이터 멤버)와 메소드 간 기능의 중간인 클래스 멤버의 특수한 유형이다.
- 프로퍼티의 읽기와 쓰기는 일반적으로 조회프로퍼티(getter)와 수정자프로퍼티(setter) 메소드 호출로 변환된다.



<br></br>
<br></br>


## 느낀점 😌

### @EqualsAndHashCode<a name="EqualsAndHashCode"></a>
- `equals`, `hashCode` 자동 생성해주는 어노테이션
- 인터넷에서 equals, hashCode의 대해 찾아보면
  - equals가 같은면  두 객체의 내용이 같다.??? 다르면, 두 객체의 내용은 다르다.????
  - hashCode가 같으면,  두 객체가 같거나 다를수 있다.??? 다르면, 두 객체는 다르다.
      
#### hashCode()
 - hashCode()로 반환되는 값 확인해보기
```java
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData){
        HelloData helloData2 = new HelloData();
        helloData2.setUsername("hello");
        helloData2.setAge(20);
        int code1 = helloData.hashCode();  //99166983
        int code2 = helloData2.hashCode(); //99166983
        System.out.println(helloData.equals(helloData2)); // true
    return "ok";
}
```
- hashCode()의 메소드를 보면 반환값이 integer값이다.
- hashMap의 key값을 구분할 때 hashcode를 기반으로 구분한다고한다.
    - 즉, 해당 객체를 HashMap과 같은 자료구조에서 사용한다면, 어노테이션이 필요하다.
- 아래 사진과 같이, 인스턴스주소값은 다르지만, hashcode는 같음을 알 수 있다.  
  <img src="https://user-images.githubusercontent.com/104331549/181425009-d49a12f7-2e7e-42db-bac4-8abb02bdc6b0.png" >


- hashcode() 메소드 내부코드
  ```java
   public int hashCode() {
           int PRIME = true;
           int result = 1;
           result = result * 59 + this.getAge();
           Object $username = this.getUsername();
           result = result * 59 + ($username == null ? 43 : $username.hashCode());
           return result;
           }
   }
   ```
    - 여기서 알 수 있는건, 객체 내부의 있는 값만 다룬다는 점이다. 
    - 그래서 한가지 실험을 해보았다.

  <br></br>

> 만약, 동일한 필드값(내부의 값)을 가지는 객체 2개를 만들고   
> 각각, 인스턴스를 만드는데, 내부의 값이 같다면? 어떻게 될까?

- HelloData.java
```java
@Data
public class HelloData {
    private String username;
    private int age;
}
```
- HelloData2.java
```java
@Data
public class HelloData2 {
    private String username;
    private int age;
}
```

```java
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(HelloData helloData){
    HelloData2 helloData2 = new HelloData2();
    helloData2.setUsername("hello");
    helloData2.setAge(20);
    int code1 = helloData.hashCode();
    int code2 = helloData2.hashCode();
    System.out.println(code2 == code1); // true
    System.out.println(helloData2.equals(helloData)); // false
    return "ok";
}
```

<img src="https://user-images.githubusercontent.com/104331549/181428526-d3db79a3-a7d3-469f-9e49-51168fd54c5b.png">

- 원하는대로, hashcode의 값은 서로 일치하였으나, 동일한 객체임을 확인하는 `equals()`에서는 `false`가 나온다는 것을 알 수 있었다. 
  - 결국 객체 내부의 값이 같은지 확인하는 값이 `hashcode()`이며,
  - 같은 객체인지 확인하는 값이 `equals()`라는 것을 깨닫는 시간이었다.
- 내부 필드값이 동일하다면, hashcode값도 계속해서 동일함을 알 수 있었다.
- hashMap<Integer, String>과 같은 hashMap의 key값을 구분할 때, 
  - 같은 `Integer타입`임에도 내부 값이 다름으로, hashcode를 기반으로 구분하여, 좋은 성능을 낼 수 있다고 한다.

### 참고 링크
- [LocaleResolver 참고링크](https://devbox.tistory.com/entry/Spring-Locale-%EC%B2%98%EB%A6%AC)
- [hashCode에 대하여](https://brunch.co.kr/@mystoryg/133)