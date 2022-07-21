# 4. 스프링 MVC 프레임워크 만들기

- 지난 시간
  
  - 서블릿
    
    - 동적인 html 만들기 힘듬
  
  - JSP
    
    - UI 코드와 비즈니스 코드가 한 파일에 모두 담겨있음
  
  - MVC 패턴
    
    - Model: 처리된 데이터를 보관 역할
    
    - View: UI 로직 역할
    
    - Controller: 요청을 받고 처리하는 역할
    
    - 한계점
      
      - 각각의 컨트롤러에 공통적으로 들어가는 코드가 많다
        
        - JSP로 포워드 관련 코드
        
        - ViewPath 중복
      
      - 필요없는 코드도 있다.
        
        - HttpServletResponse
    
    - 해결책
      
      - 공통 기능을 처리해주는 무엇가가 필요하다 --> 프론트 컨트롤러(Front Controller)
      
      - 컨트롤러 호출 전 프론트 컨트롤러에서 공통 기능 처리
      
      - 실제로 스프링 MVC에서는 프론트 컨트롤러가 존재한다.

## 프론트 컨트롤러 도입 전

<img width="533" alt="image" src="https://user-images.githubusercontent.com/49191949/180020232-5a999a17-fca3-4438-bb58-9d253dee4cb1.png">

## 프론트 컨트롤러 도입 후

<img width="539" alt="image" src="https://user-images.githubusercontent.com/49191949/180020387-c2e88b83-f362-4a4a-b4d1-18790e1628e4.png">

- 프론트 컨트롤러 패턴 특징
  
  - 프론트 컨트롤러가 **모든 요청을 받는다.**
  
  - 프론트 컨트롤러는 **요청에 맞는 컨트롤러를 찾은 다음에 호출**해준다.
  
  - 프론트 컨트롤러만 서블릿을 사용하고 **나머지 컨트롤러는 서블릿를 사용하지 않아도 된다.**
  
  - **스프링 웹 MVC의 DispatcherServlet**이 프론트 컨트롤러 패턴으로 구현되어 있다.

## 버전1 - 프론트 컨트롤러 도입하기

<img width="519" alt="image" src="https://user-images.githubusercontent.com/49191949/180021306-f545f230-fe4f-4209-80a9-90dd62460226.png">

- 프론트 컨트롤러 역할
  
  - 1.URL매핑 정보를 사용해서 **컨트롤러를 조회**하기
  
  - 2.**URL에 맞는 컨트롤러 호출**해주기
  
  - 3.모든 **요청을 받음**

##### 버전1 프론트 컨트롤러

```java
@WebServlet(name = "frontControllerServletV1", urlPatterns = "/front-
controller/v1/\*") // 3.요청 받기 
public class FrontControllerServletV1 extends HttpServlet {

private Map<String, ControllerV1> controllerMap = new HashMap<>();

public FrontControllerServletV1() {
          controllerMap.put("/front-controller/v1/members/new-form", new
  MemberFormControllerV1());
          controllerMap.put("/front-controller/v1/members/save", new
  MemberSaveControllerV1());
          controllerMap.put("/front-controller/v1/members", new
  MemberListControllerV1());
}

@Override
      protected void service(HttpServletRequest request, HttpServletResponse
  response)
              throws ServletException, IOException {
          System.out.println("FrontControllerServletV1.service");
          String requestURI = request.getRequestURI();    // 컨트롤러 조회
          ControllerV1 controller = controllerMap.get(requestURI);
          if (controller == null) {
              response.setStatus(HttpServletResponse.SC_NOT_FOUND);
return; }
          controller.process(request, response);    // 컨트롤러 호출
      }
}
```

- 문제점
  
  - 프론트 컨트롤러를 도입만 했을 뿐, 아직 각각의 컨트롤러에는 **forward와 viewPath가 중복**되어 있고, 깔끔하지 않다.
  
  - ```java
    String viewPath = "/WEB-INF/views/new-form.jsp";
    RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
    dispatcher.forward(request, response);
    ```
  
  - 이 문제를 해결하기 위해 별도의 뷰를 처리하는 객체를 만들자

## 버전2 - 프론트 컨트롤러

<img width="522" alt="image" src="https://user-images.githubusercontent.com/49191949/180023949-e7132aff-b700-4bdb-9a60-981eefb59f87.png">

##### MyView.java

```java
public class MyView {
      private String viewPath;

      public MyView(String viewPath) {
          this.viewPath = viewPath;
      }
      public void render(HttpServletRequest request, HttpServletResponse
      response) throws ServletException, IOException {
           RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
           dispatcher.forward(request, response); // forward 공통 처리 
      }
}
```

- viewPath 정보를 갖고 있음

- render 메서드를 사용하면 JSP로 이동 가능

##### ControllerV2 인터페이스

```java
 public interface ControllerV2 {
        // View 반
      MyView process(HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException;    // MyView를 반환
  }
```

- MyView를 반환함

### 그래서 버전 1과 뭐가 달라졌나?

- 버전 1의 컨트롤러에는 뷰로 이동하는 코드가 중복되어 있었는데, 이제 그 역할을 프론트 컨트롤러에서 하기 때문에 프론트 컨트롤로 외 컨트롤러에 뷰로 이동하는 중복되는 코드가 제거되어 문제가 해결됨. 프론트 컨트롤러는 뒤 컨트롤러에서 반환받은 View 객체를 이용해서 forward 로직을 수행한다.

- ```java
  public class MemberFormControllerV2 implements ControllerV2 {
  @Override
        public MyView process(HttpServletRequest request, HttpServletResponse
        response) throws ServletException, IOException {
            return new MyView("/WEB-INF/views/new-form.jsp");
        }
  }
  ```

### 버전 2에서 남아있는 문제점

- **서블릿(HttpServletRequest, HttpServletResponse) 종속성**
  
  - 컨트롤러는 굳이 서블릿을 사용할 필요는 없다. 
  
  - 서블릿을 받으면 서블릿에 의존하게 되고, 테스트 코드 작성도 어렵다는 단점이 있다.
  
  - 요청 파라미터 정보는 Map을 이용해서 넘겨줄 수 있다.
  
  - 그리고 request 객체를 Model로 사용하는 대신 별도의 Model 객체를 만들면 된다.
  
  - 서블릿 종속성을 제거하자

- **뷰 이름 중복**
  
  - 컨트롤러에서 설정하는 뷰 이름에 중복이 있다.
  
  - 접두어 중복: `/WEB-INF/views/`
  
  - 접미어 중복:`.jsp`
  
  - 이러한 중복은 뷰 폴더 위치가 변경되면 모두 수정해야 한다
  
  - <img width="317" alt="image" src="https://user-images.githubusercontent.com/49191949/180027407-da0e85e2-8df1-41a3-a970-540e7c031344.png">
  
  - 컨트롤러는 뷰의 물리 이름을 반환하지 말고 **논리 이름(변화하는 부분)을 반환**하도록 하자

## 버전 3

![](/Users/kchs94/Library/Application%20Support/marktext/images/2022-07-21-00-55-27-image.png)

- 3. 컨트롤러는 MyView(물리주소)가 아닌 ModelView(논리주소)를 반환

- 4.프론트 컨트롤러는 viewResolver를 호출
  
  - 논리 주소를 이용해서 물리 주소를 갖고있는 MyView를 얻음

- 5.뷰 리졸버는 MyView 반환 

- 6.render(model) 호출

#### ModelView는 뭘까

- Model을 사용하기 위해 request.setAttribute()를 이용했는데, 이러한 서블릿 종속성을 벗어나기 위해 직접 만든 Model이다.

- Model 역할과 논리 주소를 저장한다.

- ```java
  @Getter @Setter
  public class ModelView {
        private String viewName;    // 뷰 논리 이름 
        private Map<String, Object> model = new HashMap<>();    // Model
  }
  ```

- 

##### ControllerV3

```java
public interface ControllerV3 { // 매개변수에 서블릿 없음
      ModelView process(Map<String, String> paramMap);    //ModelView 반
  }
```

- 이 인터페이스를 구현한 컨트롤러는 이제 서블릿 기술을 전혀 사용하지 않으므로 서블릿의 종속성에서 벗어나게 된다. 따라서 버전2의 문제점1인 서블릿 종속성을 해결한다.

- 프론트 컨트롤러에서 HttpServletRequest가 제공하는 정보를을 paramMap에 담아 인자로 넘겨주면 된다.

- 그 결과 컨트롤러는 뷰의 논리적 이름과 Model 데이터를 포함하는 ModelVIew 객체를 반환한다

##### 예시: Save 컨트롤러

```java
public class MemberSaveControllerV3 implements ControllerV3 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override                            // 서블릿 없
    public ModelView process(Map<String, String> paramMap) {
          String username = paramMap.get("username");
          int age = Integer.parseInt(paramMap.get("age"));
          Member member = new Member(username, age);

          memberRepository.save(member);
          ModelView mv = new ModelView("save-result");//뷰 논리 이름 + 모델 생성

          mv.getModel().put("member", member); // Model 역할
          return mv;    // ModelView 반
    }
}
```

### v3 - 프론트 컨트롤러

```java
@Override
protected void service(HttpServletRequest request, HttpServletResponse
response)
            throws ServletException, IOException {

        String requestURI = request.getRequestURI();
        ControllerV3 controller = controllerMap.get(requestURI); // 1.컨트롤러 조회 
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
return; }
        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap); // 2. 컨트롤러 호출 3. ModelView 반
        String viewName = mv.getViewName();
        MyView view = viewResolver(viewName); // 4.5. 뷰 리졸버 호출 및 MyView 반환 
        view.render(mv.getModel(), request, response); // 6. 렌더 기능 호출 
    }
    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName,
request.getParameter(paramName)));
           return paramMap;
      }

      private MyView viewResolver(String viewName) {
          return new MyView("/WEB-INF/views/" + viewName + ".jsp");
}    
```

- 뷰 리졸버
  
  - 논리 뷰 이름을 입력으로 받아 실제 물리 뷰 경로를 담고있는 MyView를 반환한다.
  
  - `view.render(mv.getModel(), request, response)`
    
    - JSP는 `request.getAttribute()`로 데이터를 조회하기 때문에, 모델에서 데이터를 꺼낸후 request.setAttribute()로 담아줘야 한다.
    
    - ```java
       public class MyView {
      
            private String viewPath;
      
            public MyView(String viewPath) {
                this.viewPath = viewPath;
      }
            public void render(HttpServletRequest request, HttpServletResponse
            response) throws ServletException, IOException {
                RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
                dispatcher.forward(request, response);
            }
      
            public void render(Map<String, Object> model, HttpServletRequest request,
            HttpServletResponse response) throws ServletException, IOException {
                modelToRequestAttribute(model, request);
                RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
                dispatcher.forward(request, response);
           }
      
            private void modelToRequestAttribute(Map<String, Object> model,
            HttpServletRequest request) {
                model.forEach((key, value) -> request.setAttribute(key, value));
            }
      }
      ```

## 버전 4 - 단순하고 실용적인 컨트롤러

- 앞선 리팩토링 v3으로 컨트롤러는 잘 설계되었다고 말할 수 있다.
  
  - 서블릿 종속성 제거
  
  - 뷰 경로 중복 제거

- 하지만 컨트롤라 안에 귀찮은 일이 남아있다. 좋은 프레임워크는 개발자가 편리하게 사용 할 수 있어야한다.
  
  - 컨트롤러 안에서 ModelView 객체를 생성하고 반환해야하는 점은 귀찮다..! 
  
  - 컨트롤러 안에서 모델을 만드는 것은 귀찮으니 아예 모델을 프론트 컨트롤러에서 만들고 컨트롤러를 호출할 때 인자로 넘겨주자!
  
  - 그러면 뷰 논리 이름만 반환하면 된다..!

<img width="524" alt="image" src="https://user-images.githubusercontent.com/49191949/180035651-645df612-0508-4a8a-9be6-9a73183348c8.png">

##### Controller4 인터페이스

```java
 public interface ControllerV4 {
      /**
       * @param paramMap
       * @param model
       * @return viewName
       */        // 모델까지 프론트 컨트롤러에서 만들어서 인자로 넣어
      String process(Map<String, String> paramMap, Map<String, Object> model);
  }
```

##### MemberSaveControllerV4

```java
public class MemberSaveControllerV4 implements ControllerV4 {

      private MemberRepository memberRepository = MemberRepository.getInstance();

      @Override
      public String process(Map<String, String> paramMap, 
        Map<String, Object> model) { // model 객체를 인자로 받음
          String username = paramMap.get("username");
          int age = Integer.parseInt(paramMap.get("age"));
          Member member = new Member(username, age);
          memberRepository.save(member);
          model.put("member", member);
          // 모델 객체를 생성하는 코드가 사라짐
          return "save-result"; // 뷰 논리 이름 반환 
} }
```

##### MemberSaveControllerV4

```java
public class MemberSaveControllerV4 implements ControllerV4 {

      private MemberRepository memberRepository = MemberRepository.getInstance();

      @Override
      public String process(Map<String, String> paramMap,
                 Map<String, Object> model) {    // 인자로 model 받음 

          String username = paramMap.get("username");
          int age = Integer.parseInt(paramMap.get("age"));
          Member member = new Member(username, age);
          memberRepository.save(member);
          model.put("member", member);
          return "save-result";    // 뷰 논리 이름 반환 
} }
```

##### FrontControllerServletV4

```java
public class FrontControllerServletV4 extends HttpServlet {

    private Map<String, ControllerV4> controllerMap = new HashMap<>();

    public FrontControllerServletV4() {
        controllerMap.put("/front-controller/v4/members/new-form", new
            MemberFormControllerV4());
        controllerMap.put("/front-controller/v4/members/save", new
            MemberSaveControllerV4());
        controllerMap.put("/front-controller/v4/members", new
            MemberListControllerV4());
    }

    @Override
    protected void service(HttpServletRequest request,
             HttpServletResponse response)
                throws ServletException, IOException {

        String requestURI = request.getRequestURI();
        ControllerV4 controller = controllerMap.get(requestURI);
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
        return; }

        Map<String, String> paramMap = createParamMap(request); 
        Map<String, Object> model = new HashMap<>(); //모델 생추가
        String viewName = controller.process(paramMap, model); // 컨트롤러에 인자로 model을 넘겨줌  반환값으로 뷰 논리 이름을 받음 
        MyView view = viewResolver(viewName);
        view.render(model, request, response);
    }
    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                   .forEachRemaining(paramName -> paramMap.put(paramName,
  request.getParameter(paramName)));
          return paramMap;
      }
      private MyView viewResolver(String viewName) {
          return new MyView("/WEB-INF/views/" + viewName + ".jsp");
        }
}
```

- 프론트 컨트롤러가 모델 객체를 생성해서 컨트롤러에게 전달

- 프론트 컨트롤러는 뷰의 논리 이름을 반환받음

## 버전 5 - 유연한 컨트롤러 1

- 이전 프론트 컨트롤러의 한계점이라기보다 조금 불편한점?
  
  - 한가지 방식의 컨트롤러 인터페이스만 사용할 수 있다.
  
  - <img width="532" alt="image" src="https://user-images.githubusercontent.com/49191949/180037773-43b47092-c46f-4772-982b-130699f3fe6c.png">
  
  - ControllerV3(110v), ControllerV4(220v)를 둘 다 사용할 수 없었다
  
  - 다양한 방식의 컨트롤러를 처리하고 싶다

- 해결방법
  
  - 어댑터 패턴

### V5 구조

<img width="519" alt="image" src="https://user-images.githubusercontent.com/49191949/180038151-3eebcea2-a92c-4b20-94f6-5549ab8a091d.png">

- **핸들러 어댑터**: 어댑터 역할을 해서 다양한 종류의 컨트롤러를 호출 할 수 있도록 한다.

- **핸들러**(hanlder): 컨트롤러 이름을 더 넒은 범위의 핸들러로 변경한다. 왜냐하면 어댑터가 있기 때문에 컨트롤러 개념 뿐만아니라 어떤한 것이든 해당하는 종류의 어댑터만 있으면 다 처리할 수 있기 때문이다.

- 2.프론트 컨트롤러는 핸들러를 처리할 수 있는 핸들러 어댑터를 조회해야한다.

- 3.프론트 컨트롤러는 handle(handler) 메서드를 이용해서 핸들러 어댑터를 호출한다

- 4.핸들러 어댑터는 핸들러(컨트롤러)를 호출한다

- 5.핸들러 어댑터는 ModelView를 반환한다.

##### MyHandlerAdpater 인터페이스

- 어댑터의 스펙, 표준

- ```java
  public interface MyHandlerAdapter {
  
        boolean supports(Object handler);
  
        ModelView handle(HttpServletRequest request, HttpServletResponse response,
            Object handler) throws ServletException, IOException;
    }
  ```

- `boolean supports(Object handler)`
  
  - handler(컨트롤러)를 입력받는다.
  
  - 어댑터가 입력받은 컨트롤러를 처리할 수 있으면 참, 처리할 수 없으면 거짓

- `ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler)` 
  
  - 어댑터는 실제 4.컨트롤러를 호출하고, 5.그 결과로 ModelView를 반환한다.
  
  - 실제 컨트롤러가 ModelView를 반환하지 못하면, 어댑터가 직접 ModelView를 생성해서라도 반환한다.
  
  - 이전에는 프론트 컨트롤러에서 실제 컨트롤러를 호출했지만, 이제는 이 어댑터를 통해서 실제 컨트롤러가 호출된다.

##### ControllerV3HandlerAdapter 어댑터 구현 클래스

- ```java
  public class ControllerV3HandlerAdapter implements MyHandlerAdapter {
        @Override
        public boolean supports(Object handler) { // 해당 컨트롤러를 처리할 수 있는
            return (handler instanceof ControllerV3);
        }
  
        @Override
        public ModelView handle(HttpServletRequest request, HttpServletResponse
        response, Object handler) {
  
            ControllerV3 controller = (ControllerV3) handler;
            Map<String, String> paramMap = createParamMap(request);
  
            ModelView mv = controller.process(paramMap);
             return mv; 
        }
  
        private Map<String, String> createParamMap(HttpServletRequest request) {
            Map<String, String> paramMap = new HashMap<>();
            request.getParameterNames().asIterator()
                    .forEachRemaining(paramName -> paramMap.put(paramName,
                    request.getParameter(paramName)));
            return paramMap;
        }
  ```

##### FrontControllerServletV5 - 프론트 컨트롤러

- ```java
  @WebServlet(name = "frontControllerServletV5", urlPatterns = "/front-
  controller/v5/\*")
  public class FrontControllerServletV5 extends HttpServlet {
      // 핸들러 매핑 정보 저
      private final Map<String, Object> handlerMappingMap = new HashMap<>();   
      private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();
  
      public FrontControllerServletV5() {    // 생성자
          initHandlerMappingMap();    // 핸들러 매핑 초기화
          initHandlerAdapters();    // 핸들러 어댑터 초기화 
      }
  
      private void initHandlerMappingMap() {
          handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new
              MemberFormControllerV3());
          handlerMappingMap.put("/front-controller/v5/v3/members/save", new
              MemberSaveControllerV3());
          handlerMappingMap.put("/front-controller/v5/v3/members", new
              MemberListControllerV3()); // v4도 있다고 가정
      }
  
      private void initHandlerAdapters() {
          handlerAdapters.add(new ControllerV3HandlerAdapter()); // v4도 있다고 가정
      }
  
      @Override
      protected void service(HttpServletRequest request, HttpServletResponse
              response) throws ServletException, IOException {
  
          Object handler = getHandler(request); // 핸들러 조
          if (handler == null) {
              response.setStatus(HttpServletResponse.SC_NOT_FOUND);
  return; }
  
          MyHandlerAdapter adapter = getHandlerAdapter(handler); // 어댑터 조회 
          ModelView mv = adapter.handle(request, response, handler);// 어댑터 호출 
          MyView view = viewResolver(mv.getViewName());
          view.render(mv.getModel(), request, response);
      }
  
      private Object getHandler(HttpServletRequest request) {
          String requestURI = request.getRequestURI();
          return handlerMappingMap.get(requestURI);
      }
  
      private MyHandlerAdapter getHandlerAdapter(Object handler) {
          for (MyHandlerAdapter adapter : handlerAdapters) {
              if (adapter.supports(handler)) {
                  return adapter;
              } 
          }
               throw new IllegalArgumentException("handler adapter를 찾을 수 없습니다. handler=" + handler);
      }
        private MyView viewResolver(String viewName) {
            return new MyView("/WEB-INF/views/" + viewName + ".jsp");
      } }
  ```

##### ControllerV4HandlerAdpater 구현 클래스

```java
public class ControllerV4HandlerAdapter implements MyHandlerAdapter {
    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV4);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse
            response, Object handler) {
        ControllerV4 controller = (ControllerV4) handler;
        Map<String, String> paramMap = createParamMap(request);
        Map<String, Object> model = new HashMap<>();
        String viewName = controller.process(paramMap, model);
        ModelView mv = new ModelView(viewName); // 어댑터가 ModelView 생성해줌
        mv.setModel(model);
        return mv; } // 결국 ModelView 반환 

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName,
                request.getParameter(paramName)));
         return paramMap;
    }
```

## 각 버전별 리팩토링 정리

- 이전 MVC 패턴 한계점
  
  - 컨트롤러에서 공통화 할 수 있는 중복 코드가 있었음. 그래서 공통 처리는 프론트 컨트롤러를 도입해서 해결하도록 목표 설정

- v1: 프론트 컨트롤러 도입
  
  - 기존 구조를 유지하면서 프론트 컨트롤러 도입

- v2: View 분류
  
  - 단순히 반복되는 뷰 로직을 분리함

- v3: Model을 따로 만듬
  
  - 서블릿 종속성을 제거하기 위해
  
  - 뷰 이름 중복 문제점을 제거하기 위해

- v4: 좀 더 개발자에게 편리한 컨트롤러를 만들기 위해
  
  - 컨트롤러가 ModelView를 직접 생성하지 않도록 프론트 컨트롤러가 만들어주도록 리팩토링

- v5: 유연한 컨트롤러를 위해 어댑터 도입
  
  - 어댑터를 추가해서 다양한 컨트롤러 스펙을 받을 수 있도록 리팩토링
  
  - 다형성과 어댑터 덕분에 기존 구조를 유지하면서 프레임워크 기능을 확장할 수 있음

- 지금까지 우리가 만든 코드가 간소화한 스프링 MVC 프레임워크라고 할 수 있다.
