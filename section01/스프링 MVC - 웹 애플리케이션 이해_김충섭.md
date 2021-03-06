# 웹 애플리케이션 이해

## 웹 서버(Web Server)와 웹 애플리케이션 서버(WAS)

- **웹서버(Web Server)**
  
  - **정적 리소스**를 제공하는 서버
    
    - 정적 리소스: HTML, CSS, JS, 이미지 , 영상 등
  
  - 대표적인 웹서버: **nginx**, apache

- **웹 애플리케이션 서버**
  
  - 프로그램 코드를 실행해서 **애플리케이션 로직**을 수행하는 서버
    
    - 동적 HTML, HTTP API
    
    - 서블릿, JSP, 스프링 MVC
  
  - 대표적인 WAS: **톰캣**(Tomcat), Jetty,
  
  - 참고로 WAS는 웹서버 기능도 가진다.

- Web Server vs Web Application Server
  
  - **크게보면 웹 서버는 정적 리소스를 제공하는 서버이고, WAS는 애플리케이션 로직을 실행하는 서버**

## 여러가지 웹 시스템 구조

### WAS + DB



<img width="616" alt="image" src="https://user-images.githubusercontent.com/49191949/178232695-cb708fe7-9687-4b58-a50d-75a20c571fd5.png">



- WAS
  
  - 정적 리소스 제공 역할
  
  - 애플리케이션 로직 실행 역할
  
  - DB 접근 역할 등

- DB
  
  - 데이터 저장 역할

- 단점
  
  - **WAS가 너무 많은 역할**을 해서 과부화가 걸릴 수 있는 가능성이 있다.
  
  - 과부하가 걸려 서버가 죽으면 **오류 화면(정적리소스)도 제공할 수 없다.**

### 웹서버 + WAS + DB



<img width="782" alt="image" src="https://user-images.githubusercontent.com/49191949/178232779-36640c74-64bb-4efe-bcbc-126af9e9f547.png">



- 웹서버
  
  - 정적 리소스 제공
  
  - 동적인 처리가 필요한 경우 WAS에게 요청

- WAS
  
  - 애플리케이션 로직 처리

- DB
  
  - DB 역할

- 장점
  
  - 상황에 따른 **서버 증설을 효율적**으로 할 수 있다.
    
    - 정적 리소스가 많이 사용되면 웹서버만 증설
    
    - 애플리케이션 리소스가 많이 사용되면 WAS만 증설
  
  - WAS 서버가 죽어도 **웹서버에서 오류페이지를 제공**할 수 있다.
    
    - 참고로 웹서버는 잘 안죽고, WAS는 잘 죽는다. 
    
    - 애플리케이션 로직 처리가 더 리소스를 많이 사용하기 때문이다.

## 서블릿

- WAS에 서블릿이 없다면?
  
  
  
  <img width="285" alt="image" src="https://user-images.githubusercontent.com/49191949/178232908-7256904d-3a62-4617-b74f-b2141183a638.png">
  
  
  
  - TCP 연결, HTTP 요청 메세지 파싱해서 읽기, ..., 
  
  - 비즈니스 로직 실행
    
    - 데이터베이스에 저장 요청
  
  - Http 응답 메세지 생성, startline 작성, header 생성, 메세지 바디에 html작성 ...
  
  - **비즈니스 로직 실행이외에 반복되는 과정이 너무 길고 복잡하다.**

- 서블릿이 있는 WAS
  
  - 요청 정보는 HttpServletRequest 객체를 통해 얻는다.
  
  - 응답 정보는 HttpServletResponse로 만든다.
  
  - 개발자는 비즈니스 로직에 집중할 수 있다.

- HTTP 요청과 응답 흐름
  
  - Http 요청
  
  - WAS가 Request, Response 객체를 새로 만들어서 서블릿 객체를 호출
  
  - 개발자는 Request 객체를 이용해서 HTTP 요청 정보를 쉽게 사용
  
  - 개발자는 Response 객체를 이용해서 HTTP 응답 정보를 쉽게 입력
  
  - WAS는 Response 객체를 이용해서 HTTP 응답 메세지 생성



<img width="893" alt="image" src="https://user-images.githubusercontent.com/49191949/178234865-df81af1c-6989-4c1d-9f31-53d8f3ad9278.png">



- **서블릿 컨테이너**
  
  - 서블릿 컨테이너란 **서블릿을 지원하는 WAS**를 말한다.
    
    - 대표적인 서블릿 컨테이너: **톰캣**
  
  - 서블릿 컨테이너는 **서블릿 객체의 생성, 초기화, 호출, 종료 등 생명주기를 관리**한다.
  
  - 서블릿 객체는 **싱글톤**으로 관리된다.
    
    - 매 요청마다 객체를 생성하는 것은 비효율
    
    - 최초 **로딩 시점에 미리 서블릿 객체를 생성**
    
    - 모든 요청은 동일한 서블릿 객체에 접근
    
    - 그러므로 **공유 변수 사용에 주의**
    
    - 서블릿 컨테이너 종료시 함께 종료
  
  - **동시 요청을위한 멀티 쓰레드 처리를 지원**한다.



## 멀티 쓰레드(Multi Thread)

- 쓰레드: 애플리케이션 코드를 실행해주는 것

- 한번에 하나의 코드 라인만 수행한다.

- 만약 **동시 처리가 필요하면 쓰레드를 추가로 생성**해야 한다.



### 단일 요청 - 쓰레드 하나

- 쓰레드를 할당한다

- 할당된 쓰레드가 서블릿 객체를 호출한다.

- 서블릿 객체를 이용해서 응답 메세지를 만든다



### 다중 요청 - 쓰레드 하나



<img width="740" alt="image" src="https://user-images.githubusercontent.com/49191949/178233378-6117d84f-6404-46ac-8c1a-fd25f03e66fa.png">



- 요청1을 처리하기 위해 쓰레드가 할당되었으나 처리 지연 중

- 요청2는 쓰레드를 대기한다

- 단점
  
  - 처리가 지연되면 결국 두 요청 모두 처리 지연



### 다중 요청 - 요청마다 쓰레드 생성



<img width="758" alt="image" src="https://user-images.githubusercontent.com/49191949/178233453-bd32b81c-010a-4ac4-8fa8-2f8afb7c1f94.png">



- 요청마다 쓰레드를 생성한다.

- 장점
  
  - 요청1이 처리가 지연되어도, 요청2는 다른 쓰레드를 사용하여 응답할 수 있다.
  
  - 리소스(CPU, 메모리)가 허용될 때까지 처리 가능
  
  - 결국 동시 요청을 처리할 수 있다.

- 단점
  
  - 요청마다 쓰레드를 생성하니 비싸고, 요청을 받고 생성하니 **응답속도가 느리다.**
  
  - 쓰레드가 번갈아가면서 리소스를 사용하니 **컨텍스트 스위칭 비용**이 발생한다
  
  - 쓰레드 생성에 제한이 없어, 요청이 리소스 허용수준보다 많이 오면 **서버가 죽을 수 있다.**

- 해결방법
  
  - 스레드 풀(Thread pool)



### 쓰레드 풀(Thread Pool)



<img width="748" alt="image" src="https://user-images.githubusercontent.com/49191949/178233557-fe6c9350-d0a7-437d-82bd-fa958e93bec5.png">



# <img width="762" alt="image" src="https://user-images.githubusercontent.com/49191949/178233659-9df56e0f-6755-407a-ab90-2e2a9970b522.png">



- **쓰레드를 미리 생성하여 보관하고 관리**한다.

- 사용 방법
  
  - 쓰레드가 필요하면 스레드 풀에서 꺼내서 사용한다
  
  - 쓰레드 사용이 끝나면 쓰레드 풀에 반납한다.
  
  - 최대 쓰레드를 모두 사용중이라서 스레드 풀에 스레드가 없으면?
    
    - 요청을 특정 숫자만큼 대기하거나, 거절하도록 설정할 수 있다

- 장점
  
  - 스레드를 미리 생성하고 반납하므로 리소스 낭비를 줄일 수 있고, **응답이 빠르다**
  
  - 대기하거나 거절할 수 있으므로, 너무 많은 요청이 들어와도 서버가 죽지 않는다. 



### 스레드 풀 실무 팁

- WAS 튜닝 핵심은 **최대 쓰레드(Max Thread) 수**이다.

- 최대 쓰레드가 너무 낮게 설정하면?
  
  - 동시 요청이 많이 들어올 때 리소스는 여유롭지만, **클라이언트 응답이 지연**된다.

![](/Users/kchs94/Library/Application%20Support/marktext/images/2022-07-11-18-29-05-image.png)



- 최대 쓰레드가 너무 높게 설정하면?
  
  - 리소스 성능보다 높게 설정하면 리소스 임계점 초과로 서버가 다운될 수 있다.



### 최대 쓰레드 수는 어떻게 결정할까?

- 애플리케이션 로직 복잡도, 리소스(CPU, 메모리, IO등)을 **복합적으로 고려**한다.

- **실전과 같은 환경에서 미리 성능 테스트**를 한다.
  
  - 성능 테스트 툴: ngrinder, 아틸러스



### WAS는 멀티 쓰레드를 지원한다

- 개발자는 멀티 쓰레드 관련해서 신경을 쓰지 않아도 된다.

- 멀티 쓰레드 환경이라는 것을 생각해서 싱글톤 객체(셔블릿, 스프링 빈)을 사용할 때 주의한다.



## HTML, HTTP API, CSR, SSR



## 클라이언트가 받는 데이터 유형 3가지

1. **정적 리소스**
   
   - 정적 html 파일, css, js, 이미지 등
   
   - 주로 웹 브라우저

<img width="682" alt="image" src="https://user-images.githubusercontent.com/49191949/178233978-0573eee5-77f1-4ade-bf8a-7ca0e7bf78eb.png">



1. **동적 html**
   
   - 서버가 동적으로 html 파일을 렌더링해서 제공
   
   - SSR



<img width="811" alt="image" src="https://user-images.githubusercontent.com/49191949/178234062-be1a7e23-f834-4bfe-930f-2b6cc8915986.png">



1. **HTTP API**
   
   - http이 아닌 순수 데이터를 전달
   
   - 주로 JSON
   
   - 다양한 시스템에서 요구됨
     
     - 예) 앱 클라이언트, 웹 클라이언트(CSR), 서버 to 서버
   
   - UI는 클라이언트가 처리

<img width="822" alt="image" src="https://user-images.githubusercontent.com/49191949/178234183-a480d369-5649-4815-9be7-0e76e8ec0dfa.png">



## Server Side Rendering(SSR) vs Client Side Rendering(CSR)

- **SSR**(서버 사이드 렌더링)
  
  - **서버에서 렌더링된 html을 만들어서 브라우저에게 전달하는 방법**
  
  - 정적인 화면에 사용
  
  - 관련기술: JSP, 타임리프

- **CSR**(클라이언트 사이드 렌더링)
  
  - **클라이언트(브라우저)에서 html을 렌더링하는 방법**
  
  - 주로 동적인 화면에서 사용
  
  - 웹 환경을 마치 앱처럼 필요한 부분부분만 변경할 수 있다.
  
  - 예) 구글 지도, 구글 캘린더
  
  - 관련 기술: 리액트, 뷰



## 자바 백엔드 웹 기술의 역사 발전



### 과거 기술

- **서블릿** - 1997년
  
  - 장점: request, response 객체를 이용하여 요청, 응답 메세지 정보를 쉽게 사용할 수 있음
  
  - 단점: **HTML 생성이 어려움**

- **JSP** - 1999년
  
  - 장점: HTML 생성이 편리함
  
  - 단점: **HTML 코드에 비즈니스 코드까지 엮여있음**

- 서블릿, JSP 조합 MVC 패턴 사용
  
  - 장점: 모델, 뷰, 컨트롤러 역할을 나누어 코드를 작성해서 유지보수에 편리

- MVC 프레임워크 발전 - 2000~2010
  
  - MVC 패턴 + 복잡한 웹 기술을 편리하게 해주는 기능 제공
  
  - 스럿럿츠, 스프링 MVC(과거버전)



### 현재 기술

- **애노테이션 기반 스프링 MVC**
  
  - 가장 많이 사용하는 MVC 프레임워크
  
  - 가장 단순하고 유연함

- **스프링 부트**
  
  - WAS(톰캣)을 내장
  
  - 스프링 프로젝트 설정을 자동화



### 최신 기술

- Web 서블릿 - 스프링 MVC

- Web Reactive - Spring WebFlux
  
  - 장점
    
    - 비동기 넌 블러킹 처리
    
    - 최소 스레드로 최대 성능을 낸다. 그래서 쓰레드 컨텍스트 스위칭 비용이 적음
    
    - 함수형 스타일로 개발
    
    - 서블리 기술 사용x
  
  - 단점
    
    - 기술적 난이도 높음
    
    - RDB 지원 부족
    
    - 스프링 MVC 쓰레드 모델도 충분히 빠르다
    
    - 실무에서 거의 사용 X



## 자바 뷰 템플릿(View Template)

- **뷰 템플릿**: 서버에서 HTML을 생성해주는 기술

- JSP
  
  - 속도 느림, 기능 부족, 비즈니스 코드와 뷰와 결합

- 프리마커, 벨로시티
  
  - 속도 빠름, 다양한 기능

- **타임리프**(Thymeleaf)
  
  - 내추럴 템플릿: html의 모양을 유지함
  
  - 스프링 MVC에서 다양한 기능을 지원
  
  - 최고의 선택
