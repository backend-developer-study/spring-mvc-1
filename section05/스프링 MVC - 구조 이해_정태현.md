# 05_스프링 MVC - 구조 이해

## 스프링 MVC 전체 구조

> 이전 강의를 통해 직접 만든 MVC 프레임워크와 스프링 MVC를 비교해보자.

### 직접 만든 MVC 프레임워크 구조

<p align="center">
  <img src ="https://user-images.githubusercontent.com/52318666/180720705-51145af4-ee38-4f38-a93b-8794d54cca3e.png" >
</p>

### 스프링 MVC 구조

<p align="center">
  <img src ="https://user-images.githubusercontent.com/52318666/180721237-488053c9-38d4-4b0c-9654-936fc95d6ccb.png" >
</p>

- 직접 만든 프레임워크 구조 vs 스프링 MVC 구조
  - FrontController -> DispatcherServlet
  - handlerMappingMap -> HandlerMapping
  - MyHandlerAdapter -> HandlerAdapter
  - viewResolver -> ViewResolver
  - MyView -> View

### DispatcherServlet 구조 살펴보기

- 스프링 MVC도 프론트 컨트롤러 패턴으로 구현되어 있다.
- 스프링 MVC에서 프론트 컨트롤러에 해당 하는 것이 바로 `DispatcherServlet`이다.
- 이 `DispatcherServlet`이 스프링 MVC의 핵심이다.

#### DispatcherServlet의 다이어그램

<p align="center">
  <img src ="https://user-images.githubusercontent.com/52318666/180722394-b5d81d76-62d9-4281-aba7-92282365f781.png" >
</p>

- `DispatcherServlet`은 `HttpServlet`을 상속 받는다.
- 스프링 부트는 `DispatcherServlet`을 서블릿으로 자동 등록하면서 **모든 경로**`(urlPatterns="/")`에 대해서 매핑한다.
  - 참고) 더 자세한 경로가 우선순위가 더 높다. 그래서 기존에 등록한 서블릿도 함께 동작한다.
- 서블릿이 호출되면 `HttpServlet`이 제공하는 `service()`가 호출된다.
- 스프링 MVC는 `DispatcherSerlvet`의 부모인 `FramworkServlet`에서 `service()`를 오버라이딩했다.
- `FrameworkServlet.service()` 를 시작으로 여러 메서드가 호출되면서 `DispacherServlet.doDispatch()` 가 호출된다.

### 클라이언트의 HTTP 요청이 왔을 때 스프링의 동작 순서 

1. 핸들러 조회 : 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
2. 핸들러 어댑터 조회 : 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
3. 핸들러 어댑터 실행 : 핸들러 어댑터를 실행한다.
4. 핸들러 실행 : 핸들러 어댑터가 실제 핸들러를 실행한다.
5. ModelAndView 반환 : 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다.
6. viewResolver 호출 : 뷰 리졸버를 찾고 실행한다.
7. View 반환 : 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다.
8. 뷰 렌더링 : 뷰를 통해서 뷰를 렌더링 한다.

## 핸들러 매핑과 핸들러 어댑터

> 컨트롤러가 호출되기 위해서는 다음 2가지가 필요하다.

- HandlerMapping(핸들러 매핑)
  - 핸들러 매핑에서 컨트롤러를 찾을 수 있어야 한다.
- HandlerAdapter(핸들러 어댑터)
  - 핸들러 매핑을 통해서 찾은 핸들러를 실행할 수 있는 핸들러 어댑터가 필요하다.

> 이 핸들러 매핑과 핸들러 어댑터를 스프링 부트가 자동으로 등록해준다.
> 중요한 핸들러 매핑과 핸들러 어댑터는 다음과 같다.

### HandlerMapping

```tex
0 = RequestMappingHandlerMapping : 어노테이션 기반의 컨트롤러인 @RequestMapping에서 사용 🌟
1 = BeanNameUrlHandlerMapping : 스프링 빈의 이름으로 핸들러를 찾는다.
```

### HandlerAdapter

```tex
0 = RequestMappingHandlerAdapter : 어노테이션 기반의 컨트롤러인 @RequestMapping에서 사용 🌟
1 = HttpRequestHandlerAdapter : HttpRequestHandler 처리
2 = SimpleControllerHandlerAdapter : Controller 인터페이스(어노테이션X, 과거에 사용) 처리
```

### `@RequestMapping`

- 가장 우선순위가 높은 핸들러 매핑과 핸들러 어댑터는 `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`이다.
- `@RequestMapping`의 앞글자를 따서 만든 이름으로, 이것이 스프링에서 주로 사용하는 어노테이션 기반의 컨트롤러를 지원하는 매핑과 어댑터이다.
- 실무에서는 99.9% 이 방식의 컨트롤러를 사용한다.

## 뷰 리졸버

### 뷰 리졸버 - InternalResourceViewResolver

- 스프링 부트는 `InternalResourceViewResolver`라는 뷰 리졸버를 자동으로 등록한다.
- 이때, `application.properties`에 등록한 `spring.mvc.view.prefix`, `spring.mvc.view.suffix` 설정 정보를 사용해 등록한다.

### 스프링 부트가 자동으로 등록해주는 뷰 리졸버

- 실제로는 더 많지만, 중요한 부분은 다음과 같다.

```tex
1 = BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환한다. (예: 엑셀 파일 생성 기능에 사용)
2 = InternalResourceViewResolver : JSP를 처리할 수 있는 뷰를 반환한다.
```

## 스프링 MVC - 시작하기

### 어노테이션 정리

- `@Controller`
  - 스프링이 해당 어노테이션이 붙은 클래스를 스프링 빈으로 등록한다.
    - `@Controller` 어노테이션의 내부에 `@Component` 어노테이션이 있어서 컴포넌트 스캔의 대상이 된다.
  - 스프링 MVC에서 어노테이션 기반 컨트롤러로 인식한다.
- `@RequestMapping`
  - 클라이언트의 요청에 대해 어떤 Controller, 어떤 메서드로 처리할지를 매핑하기 위한 어노테이션이다.
  - 설정한 URL이 호출되면 그에 해당하는 메서드가 호출된다.
  - 어노테이션을 기반으로 동작하기 때문에, 메서드의 이름은 임의로 지어도 된다.
- `@ModelAndView`
  - 모델과 뷰 정보를 담아서 반환한다.

- `RequestMappingHandlerMapping`은 스프링 빈 중에서 `@RequestMapping` 또는 `@Controller` 가 클래스 레벨에 붙어 있는 경우에 매핑 정보로 인식한다.

### 스프링 MVC를 이용한 회원 저장 컨트롤러 구현 예제 코드

```java
@Controller
public class SpringMemberSaveControllerV1 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/springmvc/v1/members/save")
    public ModelAndView process(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        System.out.println("member = " + member);
        memberRepository.save(member);

        ModelAndView mv = new ModelAndView("save-result");
        mv.addObject("member", member);
        return mv;
    }
}
```

- `mv.addObject("속성 이름", 속성 값)`
  - 스프링이 제공하는 `ModelAndView	`를 통해 Model 데이터를 추가할 때는 `addObject()`를 사용하면 된다.

## 스프링 MVC - 컨트롤러 통합

> 이전 강의에서 회원 저장, 회원 조회, 회원 목록 조회 컨트롤러를 각각 만들었지만, `@RequestMapping`의 기능을 이용하면 이 컨트롤러들을 하나의 컨트롤러로 통합할 수 있다.

```java
@Controller
@RequestMapping("/springmvc/v2/members")
public class SpringMemberControllerV2 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/new-form")
    public ModelAndView newForm() {
        return new ModelAndView("new-form");
    }

    @RequestMapping("/save")
    public ModelAndView save(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        System.out.println("member = " + member);
        memberRepository.save(member);

        ModelAndView mv = new ModelAndView("save-result");
        mv.addObject("member", member);
        return mv;
    }

    @RequestMapping
    public ModelAndView members() {
        List<Member> members = memberRepository.findAll();

        ModelAndView mv = new ModelAndView("members");
        mv.addObject("members", members);
        return mv;
    }
}
```

- `@RequestMapping`의 중복된 부분을 클래스 레벨로 올리고, 고유한 URL을 가지는 부분은 메서드 레벨로 설정했다.
- 이렇게 리팩토링을 진행해보니 하나의 클래스에서 회원과 관련된 컨트롤러를 편하게 확인할 수 있고, 이후에 코드가 변경된다고 하더라도 해당 클래스만 변경하면되기 때문에 유지보수가 좀 더 편해질 것으로 예상할 수 있다.

## 스프링 MVC - 실용적인 방식

> 컨트롤러의 통합으로 좀 더 관리하기 편한 코드가 됐지만, 여기서 더 나아가 스프링은 개발자가 좀 더 편리하게 개발할 수 있도록 많은 편의기능을 제공한다.
>
> 실무에서는 다음과 같은 방식을 주로 사용해서 개발을 진행한다.

```java
/**
 * Model 도입, ViewName 직접 반환
 * @RequestParam 사용
 * @RequestMapping -> @GetMapping, @PostMapping
 */
@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @GetMapping("/new-form")
    public String newForm() {
        return "new-form";
    }

    @PostMapping("/save")
    public String save(
            @RequestParam("username") String username,
            @RequestParam("age") int age,
            Model model
    ) {
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

### V3에서 변경된 점

- Model 파라미터
- ViewName 직접 반환
- `@RequestParam` 사용
  - 스프링은 HTTP 요청 파라미터를 `@RequestParam` 으로 받을 수 있다.
  - `@RequestParam("username")`은 `request.getParameter("username")`와 거의 같은 코드라 생각하면 된다. 
  - 물론 GET 쿼리 파라미터, POST Form 방식을 모두 지원한다.
- `@RequestMapping` -> `@GetMapping`, `@PostMapping`
  - `@RequestMapping`은 URL만 매칭하는 것이 아니라, HTTP Method도 함께 구분할 수 있다.
  - 참고로 Get, Post, Put, Delete, Patch 요청 모두 해당 메서드에 맞는 어노테이션이 준비되어 있다.

