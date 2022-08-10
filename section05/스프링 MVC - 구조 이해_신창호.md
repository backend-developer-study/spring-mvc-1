
# 스프링의 강의 리뷰📽
> LoadMap Part : 스프링 MVC 패턴 1편   
> Section : 05.스프링 MVC - 구조 이해  
> CreateDate : 2022.07. 21  
> UpdateDate : 2022.07.23  

### 목차
 - [스프링 MVC 전체 구조](#TheWholeStructure) 
 - [핸들러 매핑과 핸들러 어댑터_자동등록](#MappingAndAdapter) 
 - [뷰 리졸버_자동등록](#ViewResolver)
 - 스프링 MVC 
   - [시작하기](#MVCStart) 
   - [컨트롤러 통합](#Integration)
   - [실용적인 방식](#PracticalWay)
<br></br>
### IntelliJ 단축키 및 구현
- `해당 클래스에서 마우스 오른쪽버튼` -> `Diagram` -> `Show Diagram` : 클래스간 상속관계를 알 수 있다. (UML버전)
<br></br>
<br></br>

# 1.스프링 MVC 전체 구조<a name="TheWholeStructure"></a>
> 직접만든 MVC 구조와 Spring이 지원하는 MVC 구조를 비교해보자 

<p align="center"><img src="https://user-images.githubusercontent.com/104331549/179978379-d97a1bc1-ee4f-469f-a289-cd43e1bb06e2.png" width="80%"></p>

 - 기본적인 아키텍처 구조는 동일하다. 
 - 주요하게 다른 부분은 글자색을 변경하였는데 변경된 부분은 아래와 같다. 
   - **FrontController -> DispatcherServlet**
   - handlerMappingMap -> HandlerMapping
   - MyHandlerAdapter -> HandlerAdapter
   - ModelView -> ModelAndView
   - viewResolver -> ViewResolver
   - MyView  -> View

## DispatcherServlet 
 - `org.springframework.web.servlet.DispatcherServlet`
   - 이전 섹션 참고자료 참고  
 - 스프링 MVC의 프론트컨트롤러가 디스패처 서블릿이다.(스프링MVC의 핵심)


### 서블릿 등록 
 - `DispacherServlet`도 부모 클래스에서 `HttpServlet`을 상속 받아서 사용하고, 서블릿으로 동작한다
     <p align="center"><img src="https://user-images.githubusercontent.com/104331549/179981678-79a10de5-18d9-4a88-ae1e-9d45f7d7280d.png" width="80%"></p>
 - 스프링 부트는 디스패처서블릿을 내장 톰캣을 띄우면서 자동으로 등록한다.(`@WebServlet(urlPatterns="")`과는 다른 방식의 등록)
   - 그래서 이 서블릿은 모든 경로(`urlPatterns="/"`)에 대해서 매핑한다.
   - 프론트 컨트롤러의 역할 
   - 더 자세한 경로의 우선순위가 높기때문에, 기존에 등록한 서블릿도 함께 동작한다.

### 요청 흐름
 - 서블릿이 호출되면 `HttpServlet` 이 제공하는 `serivce()` 가 호출된다
 - 스프링 MVC는 `DispatcherServlet` 의 부모인 `FrameworkServlet`에서 `service()` 를 `@Override` 해두었다.
 - `FrameworkServlet.service()` 를 시작으로 여러 메서드가 호출되면서 `DispacherServlet.doDispatch()` 가 호출된다
<p align="center"><img src="https://user-images.githubusercontent.com/104331549/179983970-f3398e69-372a-4ad7-bbba-4121d1555f69.png" ></p>


### DispacherServlet.doDispatch()
 - 디스패처서블릿의 핵심인 doDispatch()코드를 보자 
    - 예외처리, 인터셉터 기능은 제외했다. 
```java
public class dispacherServlet {
   //생략
   protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
      HttpServletRequest processedRequest = request;
      HandlerExecutionChain mappedHandler = null;
      ModelAndView mv = null;

      // 1. 핸들러 조회
      mappedHandler = getHandler(processedRequest);
      if (mappedHandler == null) {
         noHandlerFound(processedRequest, response);
         return;
      }
      // 2. 핸들러 어댑터 조회 - 핸들러를 처리할 수 있는 어댑터
      HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
      // 3. 핸들러 어댑터 실행 -> 4. 핸들러 어댑터를 통해 핸들러 실행 -> 5. ModelAndView 반환
      mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
      processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
   }

   private void processDispatchResult(HttpServletRequest request,
                                      HttpServletResponse response,
                                      HandlerExecutionChain mappedHandler,
                                      ModelAndView mv,
                                      Exception exception
   ) throws Exception {
      // 뷰 렌더링 호출
      render(mv, request, response);
   }

   protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
      View view;
      String viewName = mv.getViewName();
      // 6. 뷰 리졸버를 통해서 뷰 찾기, 7. View 반환
      view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
      // 8. 뷰 렌더링
      view.render(mv.getModelInternal(), request, response);
   }
}
```

## 주요 인터페이스 목록
 - 아래 인터페이스들은 전부 `org.springframework.web.servlet.*` 디렉토리에 있다.
   - 핸들러 매핑: `HandlerMapping`
   - 핸들러 어댑터: `HandlerAdapter`
   - 뷰 리졸버: `ViewResolver`
   - 뷰: `View`

> 스프링 MVC는 코드 분량도 매우 많고, 복잡해서 내부 구조를 다 파악하는 것은 쉽지 않다.   
> 사실 해당 기능을 직접 확장하거나 나만의 컨트롤러를 만드는 일은 없다. 왜냐하면 대부분의 기능이 이미 다 구현되어 있다.  
 - 그래도 이렇게 핵심동작을 알아두어야 향후 문제가 발생했을때, 어떤 부분에서 **문제**가 발생했는지 쉽게 **파악**할 수 있다.
 - 또한, 확장 포인트가 필요할 때도, 어떤 부분을 **확장**해야할 지 **파악** 할 수가 있다.

<br></br>
<br></br>

# 핸들러 매핑과 핸들러 어댑터<a name="MappingAndAdapter"></a>


### 과거버전 스프링 컨트롤러 
- org.springframework.web.servlet.mvc.Controller
  - 인터페이스이며, 요청과 응답을 인자로 받고, `ModelAndView`를 반환한다.
  - Controller 인터페이스는 @Controller 애노테이션과는 전혀 다르다.
```java
public interface Controller {
        ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception;
    }
```
 
### OldController
```java
@Component("/springmvc/old-controller")
public class OldController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("OldController.handleRequest");
        return null;
    }
}
```
- 실행하고, `http://localhost:8080/springmvc/old-controller`에 접속하면, 위 문장이 출력된다.

> 어떻게 이 컨트롤러가 호출되는 것일까? 

<br></br>
## OldController 호출
- `@Component` : 이 컨트롤러는 /springmvc/old-controller 라는 이름의 스프링 빈으로 등록되었다.
  - 빈의 이름으로 URL을 매핑할 것이다

<p align="center"><img src="https://user-images.githubusercontent.com/104331549/179998515-1d84d399-9cc4-4b4e-8ac6-1ebf4b45f3e4.png" width="50%"></p>

### HandlerMapping(핸들러 매핑)
 - 핸들러 매핑에서 이 컨트롤러를 찾을 수 있어야 한다
   - 스프링 빈의 이름으로 핸들러를 찾을 수 있는 핸들러 매핑이 필요하다.
   - Spring은 이미 구현되어 있어, 빈 이름으로 해당 핸들러를 찾을 수 있다.
### HandlerAdapter(핸들러 어댑터)
 - 핸들러 매핑을 통해서 찾은 핸들러를 실행할 수 있는 핸들러 어댑터가 필요하다.
   - `Controller` 인터페이스를 실행할 수 있는 핸들러 어댑터를 찾고 실행해야 한다.
   - Spring은 이미 구현되어 있어, 해당 어댑터를 찾아 실행시킬 수 있다.

<br></br>

## 스프링 부트가 **자동 등록**하는 핸들러 매핑과 핸들러 어댑터

### HandlerMapping
```text
0 = RequestMappingHandlerMapping : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용 (@RestController)
1 = BeanNameUrlHandlerMapping : 스프링 빈의 이름으로 핸들러를 찾는다.
```
### HandlerAdapter
```text
0 = RequestMappingHandlerAdapter : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용 (@RestController)
1 = HttpRequestHandlerAdapter : HttpRequestHandler 처리
2 = SimpleControllerHandlerAdapter : Controller 인터페이스(애노테이션X, 과거에 사용) 처리 
```

## 정리하자면 
 - OldController 를 실행하면서 사용된 객체는 다음과 같다.
   - `HandlerMapping` = 1. `BeanNameUrlHandlerMapping`
   - `HandlerAdapter` = 2. `SimpleControllerHandlerAdapter`
 - 하지만, 핸들러매핑과 어댑터을 대부분 `@RequestMapping` 어노테이션으로 처리한다.
   - 99.9% Controller와 `@RequestMapping` 세트

<p align="center"><img src="https://user-images.githubusercontent.com/104331549/180592287-3aeb9fab-b39d-425e-a868-7af98fe3aacd.png" width="80%"></p>

<br></br>
<br></br>

# 뷰 리졸버<a name="ViewResolver"></a>

## OldController - View 조회할 수 있도록 변경
> 2가지만 추가해주면 View를 조회할 수 가 있다.

### 컨트롤러에 반환값 추가
 - View를 사용할 수 있도록 코드 추가
 - ModelAndView 타입의 반환값만 추가해주면 된다.
```java
@Component("/springmvc/old-controller")
public class OldController implements Controller {
 @Override
 public ModelAndView handleRequest(HttpServletRequest request,
HttpServletResponse response) throws Exception {
 System.out.println("OldController.handleRequest");
 return new ModelAndView("new-form");  // 추가된 코드
 }
}
```
### application 설정 값 추가 
 view 풀네임 경로를 만들기위해 `application.properties`에 설정값을 추가해 준다.
```properties
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

### View 출력 
 - 위 2가지만 추가해주면 View가 나온다. 
 - 스프링부트는 `InternalResourceViewResolver` 라는 뷰 리졸버를 자동으로 등록
   - 이때  `application.properties` 에 등록된 설정 정보를 사용한다.

> `InternalResourceViewResolver` 이란 무엇일까?

<br></br>

## 뷰 리졸버 동작 방식
 - 다시한번 언급하지만 뷰 리졸버의 역할은 HTTP요청에 해당(혹은 수행)하는 view를 반환해주는 것이다.
 - 스프링 부트는 이러한 view를 **자동 등록**해주는 기능이 있다.

### 자동등록 우선순위 
 - (중요한 부분위주, 일부 생략)
```text
1 = BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환한다. (예: 엑셀 파일 생성 기능에 사용)
2 = InternalResourceViewResolver : forward()를 사용하는 JSP를 처리할 수 있는 뷰를 반환한다
```
### 동작 흐름
 1. 핸들러 어댑터로 논리이름을 얻은 디스패처서블릿(DispacherServlet)이 뷰리졸버(viewResolver) 호출
 2. `BeanNameViewResolver` 을 통해 빈 이름으로 뷰을 찾는다. 
 3. 동일한 빈이름이 없으니 그다음 우선순위인 `InternalResourceViewResolver`로 뷰를 찾는다.
 4. 찾은 view를 디스패처서블릿(DispacherServlet)에 반환해주고, 여기서 view.render()를 호출한다. 
 5. 여기서 view가 InternalResourceView 타입으로, forward()를 사용한다.

#### InternalResourceViewResolver
 - JSP처럼 포워드 `forward()`를 호출해서 처리할 수 있는 경우에 사용한다.
   - 내부적으로  `forward()`메소드를 사용하여 뷰를 처리함.
> 다른 뷰 템플릿들은 forward()과정없이 바로 렌더링된다. ex.Thymleaf

<p align="center"><img src="https://user-images.githubusercontent.com/104331549/180592341-cff91c18-8898-48de-9c51-08d4144c305d.png" width="80%"></p>

- 핸들러 부분 생략됨을 참고

<br></br>
<br></br>
 
# 스프링 MVC 시작하기<a name="MVCStart"></a>
> 스프링이 제공하는 애노테이션은 매우 유용하고 실용적이다.

### @RequestMapping
- 위에서 보았듯이 핸들러 매핑과 핸들러 어댑터 우선순위 1등이다. 
  - RequestMappingHandlerMapping
  - RequestMappingHandlerAdapter
> 이제 @RequestMapping으로 변경해보자

<br></br>

## @RequestMapping 적용하기
### 회원등록 폼
```java
@Controller //스프링이 자동으로 스프링 빈으로 등록
            // 컨트롤러로 인식
public class SpringMemberFormControllerV1 {

    @RequestMapping("/springmvc/v1/members/new-form")  //메서드이름은 임의로 지으면 된다.
    public ModelAndView process(){
        return new ModelAndView("new-form");
    }
}
```
 - `RequestMappingHandlerMapping`은 스프링 빈 중에서
   - @RequestMapping 또는 @Controller가 클래스 레벨에 붙어 있는 경우에
   - 매핑 정보로 인식한다. 

<p align="center"><img src="https://user-images.githubusercontent.com/104331549/180593234-7561a1eb-cad1-4267-941c-1f64b841f687.png" ></p>

```java
@Controller
@RequestMapping
public class SpringMemberFormControllerV1 {
   @RequestMapping("/springmvc/v1/members/new-form")
   public ModelAndView process() {
      return new ModelAndView("new-form");
   }
}
```
- 회원저장, 회원조회 구현은 생략

<br></br>
<br></br>

# 스프링 MVC - 컨트롤러 통합<a name="Integration"></a>
- `@RequestMapping`은 메서드 단위로 적용되기 때문에, 사용하여 하나의 클래스로 통합할 수가 있다.
## @RequestMapping으로 통합하기
### SpringMemberControllerV2
```java
@Controller
@RequestMapping("/springmvc/v2/members")
public class SpringMemberControllerV2 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/new-form")
    public ModelAndView newform() {
        return new ModelAndView("new-form");
    }

    @RequestMapping("/save")
    public ModelAndView save(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelAndView mav = new ModelAndView("save-result");
        mav.addObject("member", member);
        return mav;
    }

    @RequestMapping
    public ModelAndView members() {
        List<Member> members = memberRepository.findAll();

        ModelAndView mav = new ModelAndView("members");
        mav.addObject("members", members);
        return mav;
    }
}
```
 - `mv.addObject("member", member)`
   - 스프링에서 제공하는 ModelAndView를 통해 Model 데이터를 추가할 때는 addObject()를 사용한다. 
   - 이 데이터는 이후 뷰 렌더링 할 때 사용

<br></br>
<br></br>

# 스프링 MVC - 실용적인 방식<a name="PracticalWay"></a>
 - ModelView로만 반환하게되면, 다른 방식의 컨트롤러를 사용할 수가 없게 된다. (핸들러역할이 적용이 안됨)
 - 다양한 방식의 컨트롤러를 사용하기위해 실용적인 방식으로 바꿔보자.

```java
@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @GetMapping("/new-form")
    public String newform() {
        return "new-form";
    }

    @PostMapping("/save")
    public String save(
            @RequestParam("username") String username,
            @RequestParam("age") int age,
            Model model) {

        Member member = new Member(username, age);
        memberRepository.save(member);

        model.addAttribute("member", member);
        return "save-result";
    }

    @GetMapping
    public String members(Model model) {
        List<Member> members = memberRepository.findAll();
        model.addAttribute("members", members);
        return "members";
    }
}
```


## 바뀐 내용
###  @RequestMapping @GetMapping, @PostMapping
 - `@RequestMapping`의 경우 HTTP Method를 전부 포함하는 경우이다.
   - HTTP Method의 Get요청만 지원하는 `@GetMapping`을 사용 
   - 그래서 `@RequestMapping`은 class 에서만 사용

### ViewName 직접 반환
 - 반환값을 다양한 형태로 반환 할 수 있다.
   - 뷰 논리 이름으로 반환가능
   - Model, ModelAndView등 가능

### @RequestParam 사용
 - HTTP 요청 자체를 받을 수도 있지만, 파라미터만 따로 추출하여 받을 수 있다. 
 - 물론, GET 쿼리 파라미터, POST Form 방식을 모두 지원한다.
   - POST의 바디가 있는 경우라면, `@RequestBody`로 추출할 수 있다.

### Model 파라미터
- save() , members() 를 보면 Model을 파라미터로 받는 것을 확인할 수 있다. 스프링 MVC도 이런 편의  기능을 제공
- SSR방식으로 view를 구현한다면 Model파라미터를 적극 사용할 것 같다.



<br></br>
<br></br>

## 느낀점 😌
- 결국 Spring이 자동적으로 해주는 부분이 생각보다 크다는 것을 느꼈다. 
- 현재는 CSR방식으로 구현하는 걸 연습하고 있기에, 핸들러 매핑이랑 핸들러 어댑터의 자동등록위주로 사용하고 있찌만, 
   - 추후에 SSR방식으로 구현하게 된다면 Model 자동등록에 대해서도 spring의 혜택을 받아볼 수 있을 것 같다.

### 참고 링크

