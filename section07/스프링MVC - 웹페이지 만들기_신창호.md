
# 스프링의 강의 리뷰📽
> LoadMap Part : 스프링 MVC 패턴 1편  
> Section : 07.웹페이지 만들기  
> CreateDate : 2022.08.04  
> UpdateDate :  

### 목차
- [타임리프](#thymeleaf)
- [PRG Post/Redirect/Get](#PRG)
- [RedirectAttributes](#RedirectAttributes)

<br></br>
### IntelliJ 단축키
<br></br>
<br></br>

# 타임리프<a name="thymeleaf"></a>
### 타임리프 사용 선언
```html
<html xmlns:th="http://www.thymeleaf.org">
```

### 타임리프 핵심
- 핵심은 `th:xxx` 가 붙은 부분은 서버사이드에서 렌더링 되고, 기존 것을 대체한다.
- 실제 웹브라우저에서는 `th:`속성을 알지 못하므로, 무시해서 나오기 때문에, HTML을 파일 보기를 유지하면서 템플릿 기능도 할 수 있다.
- JSP 파일은 웹 브라우저에서 그냥 열면 JSP 소스코드와 HTML이 뒤죽박죽 되어서 정상적인 확인이 불가능하다. 오직 서버를 통해서 JSP를 열어야 한다.
  - jsp의 경우, 기존 html로만 알아보기 매우 힘듬.
- 타임리프로 사용하면, 파일의 명을 보여줄 필요없이 동적으로 설계할 수 있다.
- 게 순수 HTML을 그대로 유지하면서 뷰 템플릿도 사용할 수 있는 타임리프의 특징을 **네츄럴 템플릿(natural templates)**이라 한다

<br></br>
## 간단히 알아보기 
### 속성(내용) 변경 - th:href
- ```html
  th:href="@{/css/bootstrap.min.css}"
  ```
- HTML을 그대로 볼 때는 `href` 속성이 사용되고, **뷰 템플릿**을 거치면 `th:href` 의 값이 `href` 로 대체되면서 동적으로 변경할 수 있다
- 타임리프 뷰 템플릿을 거치게 되면 원래 값을 th:xxx 값으로 변경한다. 만약 값이 없다면 새로 생성한다.

<br></br>

### URL 링크 표현식 - `@{...}`
- @{...} : 타임리프는 URL 링크를 사용하는 경우 @{...} 를 사용한다. 이것을 URL 링크 표현식이라 한다.
- URL 링크 표현식을 사용하면 서블릿 컨텍스트를 자동으로 포함한다

#### 예시 링크표현식 1 (`th:onclick`)
```html
<button class="btn btn-primary float-end"
        onclick="location.href='addForm.html'" <!-- 기본 onclick -->
        th:onclick="|location.href='@{/basic/items/add}'|"  <!-- th:onclick -->
        type="button">상품 등록
</button>
```
#### 예시 링크표현식 2 (`th:href`)
```html
<td><a href="item.html"
       th:href="@{/basic/items/{itemId}(itemId=${item.id})}" <!-- url 링크 표현식 경로변수 사용 -->
       th:text="${item.id}">
       회원id</a></td>
<td><a href="item.html"
       th:href="@{|/basic/items/${item.id}|}"<!-- url 링크 표현식 간단히 사용 -->
       th:text="${item.itemName}">
        상품명</a></td>
```
 - URL 링크 표현식을 사용하면 경로를 템플릿처럼 편리하게 사용할 수 있다
 - 예시  
   - ```html
     <a th:href="@{/basic/items/{itemId}(itemId=${item.id}, query='test')}" ></a>
     ```
   - 생성링크 : http://localhost:8080/basic/items/1?query=test
 - 리터럴 대체 문법을 활용해서 간단히 사용할 수도 있다.

<br></br>
### 리터럴 대체 - `|...|`
- 타임리프에서 **문자와 표현식등은 분리되어 있**기 때문에 더해서 사용해야한다.
- 기존 타임리프 사용 예시 1
  - ```html
    <span th:text="'Welcome to our application, ' + ${user.name} + '!'">
    ```
- 리터럴 대체 사용 예시1
  - ```html 
    <span th:text= "|Welcome to our application, ${user.name}!|">
    ```
- 기존 타임리프 사용 예시2
  - ```html
    <button th:onclick="'location.href='+ '\' + '@{/basic/items/add} +'\'''" />
    ```
- 리터럴 대체 사용 예시2
  - ```html
    <button th:onclick="|location.href='@{/basic/items/add}'|" />
    ```

<br></br>
### 반복 출력 - `th:each`
- 타임리프에서도 반복문을 지원해주는데, `th:each` 를 사용한다.

```html
<tr th:each="item : ${items}">
    <td><a href="item.html"
           th:href="@{/basic/items/{itemId}(itemId=${item.id})}"  <!--동적인 URL 경로 -->
           th:text="${item.id}"> <!--동적인 데이터-->
           회원id</a></td>
    <td><a href="item.html"
           th:href="@{|/basic/items/${item.id}|}"
           th:text="${item.itemName}">
            상품명</a></td>
    <td th:text="${item.price}">10000</td>
    <td th:text="${item.quantity}">10</td>
</tr>
```
- 컬렉션의 수 만큼 `<tr>..</tr>` 이 하위 테그를 포함해서 생성된다

<br></br>
### 변수 표현식 - `${...}`
- ```html
  <td th:text="${item.price}">10000</td>
  ```
- 모델에 포함된 값이나, 타임리프 변수로 선언한 값을 조회할 수 있다
  - 프로퍼티 접근법을 사용한다. (`item.getPrice()`)

<br></br>

### 속성(내용) 변경 - `th:text`
- 위 반복문에서 
- ```html
  <td th:text="${item.price}">10000</td>
  <td th:text="${item.quantity}">10</td>
  ```
- 내용의 값을 th:text 의 값으로 변경한다.

<img src="https://user-images.githubusercontent.com/104331549/183821649-18d84a7d-f63b-4082-b4c3-c6eb897ae6be.png">


### 속성(내용) 변경 - `th:value`

```html
<input type="text" id="itemId" name="itemId" class="form-control"
       value="1" th:value="${item.id}"   <!--여기가 모델에 있는 정보 획득-->
       readonly>
```
- 모델에 있는 item 정보를 획득하고 프로퍼티 접근법으로 출력한다.
- `value` 속성을 `th:value` 속성으로 변경
<br></br>

### 속성 변경 - th:action
```html
<form action="item.html" th:action <!--이렇게 form태그안에 만듬-->
      method="post">
```
- HTML form에서 action 에 **값이 없으면 현재 URL에 데이터를 전송**한다.
  - form에 작성한 데이터를 전송하면 현재 url그대로 POST요청을 한다.
- 상품 등록 폼의 URL과 실제 상품 등록을 처리하는 URL을 똑같이 맞추고 HTTP 메서드로 두 기능을 구분한다.
  - 상품 등록 폼: `GET` `/basic/items/add`
  - 상품 등록 처리: `POST` `/basic/items/add`
  - 이렇게 하면 하나의 URL로 등록 폼과, 등록 처리를 깔끔하게 처리할 수 있다

<br></br>

## 리다이렉트
 - 상품 수정은 마지막에 뷰 템플릿을 호출하는 대신에 상품 상세 화면으로 이동하도록 리다이렉트를 호출
 - 스프링은 `redirect:/...` 으로 편리하게 리다이렉트를 지원한다
> 근데 왜 상품등록은 그냥 진행했는데, 수정은 리다이렉트를 사용한 것일까?? 

<br></br>
<br></br>

# PRG Post/Redirect/Get<a name="PRG"></a>
### 문제점
 - 기존 상품등록 요청을 완료하고 새로고침을 하게되면, 이 계속해서 중복 등록되는 것을 확인할 수 있다.
   - 웹 브라우저의 새로 고침은 마지막에 서버에 전송한 데이터를 다시 전송한다.
 - 즉, POST 요청으로 끝났다면 새로고침시, POST요청을 다시 수행하게 되는것이다.

### 해결책
- 새로 고침 문제를 해결하려면 상품 저장 후에 뷰 템플릿으로 이동하는 것이 아니라, 
- **상품 상세 화면으로 리다이렉트를 호출해주면 된다**
  - 이것을 PRG(Post Redirect Get)이라고 한다.
- 이후 새로고침을 해도 상품 상세 화면으로 이동하게 되므로 새로 고침 문제를 해결할 수 있다.
- 예시 
  - 기존 상품등록
    - ```java
      return "basic/item";
      ```
  - 리다이렉트 적용 
    - ```java
      return "redirect:/basic/items/" + item.getId();
      ```

> 하지만, 위 방식도 문제점이 있다.
> return String값을 `+ item.getId()` 처럼 URL에 변수를 더해서 사용하는 것은 URL 인코딩이 안되기 때문이다. 
> 그럼 어떻게 해야될까?

<br></br>
<br></br>

# RedirectAttributes<a name="RedirectAttributes"></a>
- 메소드 인자에 `RedirectAttributes` 타입을 매개변수로 넣어준다.
- RedirectAttributes 를 사용하면 URL 인코딩도 해주고,
- `pathVarible` , `쿼리 파라미터`까지 처리해준다.
  - `redirect:/basic/items/{itemId}`
  - pathVariable 바인딩: `{itemId}`
  - 나머지는 쿼리 파라미터로 처리: `?status=true`
```java
/**
 * RedirectAttributes
 */
@PostMapping("/add")
public String addItemV6(Item item, RedirectAttributes redirectAttributes) {
   Item savedItem = itemRepository.save(item);
   redirectAttributes.addAttribute("itemId", savedItem.getId()); //리다이렉트 모델에 파라미터 저장
   redirectAttributes.addAttribute("status", true); // 리다이렉트 모델에 status 저장
   return "redirect:/basic/items/{itemId}";
}
```

### 뷰템플릿 추가
```html
<!-- 추가 -->
 <h2 th:if="${param.status}" th:text="'저장 완료!'"></h2>
```
- `th:if` : 해당 조건이 참이면 실행
- `${param.status}` : 타임리프에서 쿼리 파라미터를 편리하게 조회하는 기능
  - 원래는 컨트롤러에서 모델에 직접 담고 값을 꺼내야 한다. 그런데 쿼리 파라미터는 자주 사용해서 타임리프에서 직접 지원한다.

## 느낀점 😌

### 참고 링크

