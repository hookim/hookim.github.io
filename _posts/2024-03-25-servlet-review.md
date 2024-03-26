---
layout: post
title: Servlet과 JSP를 활용한 MVC 구조
date: 2024-03-26 08:55:02
description: 자바 웹프로그래밍의 근간이 되는 서블릿을 통해 MVC 구조를 알아보자 
tags: 
categories: 백엔드
---
## 개요
서블릿과 JSP를 활용해서 차량등록 페이지를 만든다. 

## 구조설명
![architect drawio](https://github.com/hookim/hookim.github.io/assets/81621620/6bafb3be-0aed-478a-b645-fb3ade9e8930)
전체적으로 위와 같은 구조이다. 
1. 클라이언트는 웹서버에 요청을 한다. 만약 웹서버에 요청된 자원이 존재하면 그 파일을 읽어와서 전달한다. 만약 없다면 웹컨테이너로 전달한다. 
2. 웹 컨테이너에 있는 메인 서블릿에서 작업을 처리한다. 
3. 메인 서블릿은 서비스 클래스를 통해서 필요한 작업들을 처리한다. 서브 클래스 내부에서는 DAO를 사용한다.
4. DAO에서 DBUtil 기능클래스를 활용해서 디비에 접근을 한다. 
5. DB와 소통할 때 DTO라는 클래스에 값을 담아 소통한다.

### 싱글톤 패턴
서비스, DAO 클래스는 싱글톤 패턴으로 구현했다. 싱글톤패턴은 생성자를 private하게 설정하고 getInstance를 public, static 메소드를 통해서 단 한개의 인스턴스만을 활용하도록 하는 구조이다. 
```java
public class CarService {
	
	private static CarService instance = null;
	private static CarDao carDao = null;
	
	private CarService() {
		carDao = CarDao.getInstance();
	}
	
	public static CarService getInstance() {
		if(instance == null) {
			instance = new CarService();
		}
		
		return instance;
	}
}
```
외부에서 생성자를 통해 객체를 생성할 수 없고 오직 getInstance라는 메소드를 통해서만 생성해야 한다. 내부적으로는 instacne가 이미 생성됐는지 확인하고 있다면 기존에 있는 인스턴스를 리턴하는 형식이다. 


## 코드 설명
### 메인 서블릿
```java
	private void process(HttpServletRequest request, HttpServletResponse response) {
		try {
			String action = request.getParameter("action");
			if(action == null) {
				forward(request, response, "./index.jsp");
			}
			switch(action) {
			case "regist":
        ...
      }
    }
  }
```
메인 서블릿의 핵심 코드이다. doGet, doPost는 내부적으로 process 메소드를 호출한다. 가장 먼저 하는 작업은 action 패러미터값을 읽는 것이다. `request.getParameter("action")`는 url 쿼리 스트링이나 body의 값을 읽어온다. 이 action 패러미터를 통해 어떤 종류의 요청인지 파악을 했다.

맨처음 action이 `null`인 경우는 디폴트 처리를 한 것이다. `forward`를 하면 사용자에게 보이는 url을 변경하지 않고 원하는 요청 응답을 처리할 수 있다. 보안에 좋다. 

### 리스트 가져오기
```java
case "list":
    ArrayList<Car> list = carService.carList();
    request.setAttribute("list", list);
    forward(request, response, "./car/list.jsp");
    break;
```
리스트는 내부적으로 `carService`인스턴스를 활용한다. 해당 인스턴스는 결국 DB까지 도착해서 원하는 값을 가져와 리스트 형태로 응답한다. 

```java
public ArrayList carList() {
  return carDao.carList();
  
}
```
위 코드는 Service 클래스 인스턴스 내부이다. 내부적으로 carDao를 호출하고 있다. 
```java
	public ArrayList carList() {
		ArrayList<Car> list = new ArrayList<>();
		try {
			Connection conn = db.getConnection();
			String sql = "select * from cars";
			PreparedStatement stmt= conn.prepareStatement(sql);
			
			ResultSet rs = stmt.executeQuery();
			while(rs.next()) {
				Car car = new Car();
				car.setBrand(rs.getString("brand"));
				car.setModel(rs.getString("model"));
				car.setNumber(rs.getString("number"));
				car.setSize(rs.getString("size"));
				car.setPrice(rs.getInt("price"));
				
				list.add(car);
			}
			
			return list;
		}
		catch(Exception e) {
			e.printStackTrace();
			return null;
		}
		finally {
			db.close();
			
		}
	}
```
위 코드는 `carDao`내부 코드이다. 여기서 실제 DB에 접근하는 코드들이 나온다. prepared statement를 사용해서 sql injection을 방지할 수 있다. db 접근 부분이기 때문에 try-catch문을 활용해서 오류처리도 철저하게 진행했다. 값을 성공적으로 가져오면 `Car` DTO에 값을 집어넣어서 다시 리턴해준다.  

상세보기, 수정, 삭제의 경우도 다음과 같은 구조를 갖는다. 서블릿에서 서비스를 호출하고 DAO를 호출하고 DB에 접근해서 그 결과를 다시 역으로 리턴하는 과정이다.  

### 수정하기 뷰
  ```html
  <body>
    <nav>
      <h1>차량 수정 페이지</h1>
    </nav>
    <form action="main.do?action=update" method="post">
      <fieldset>
        <label> 차 번호 <input type="text" name="number" value="${number }" readonly></label> 
        <br> 
        <label> 모델 <input type="text" name="model" value="${model }"></label> 
        <br>
        <label> 가격 <input type="number" name="price" value="${price }"></label> 
        <br> 
        <label> 브랜드 <input type="text" name="brand" value="${brand }"></label> 
        <br> 
        <label> 차량 크기 
          <select name="size">
            <option value="소형">소형</option>
            <option value="중형">중형</option>
            <option value="대형">대형</option>
        </select>
        </label> <br> <input type="submit" value="수정"> <br> <a
          href="main.do?action=search&number=${number }">상세 페이지</a>
      </fieldset>
    </form>
  </body>
  ```
  차의 경우 차번호는 db의 primary key다. 따라서 수정이 일어나서는 안된다. 값이 수정되지 않도록 html readonly 속성을 줬다. 그러면 해당 input 값은 수정이 되지 않는다. 

### 로그인 
```java
case "login":
				String id = request.getParameter("id");
				String password = request.getParameter("password");
				String remember = request.getParameter("remember");
				
				Member member = new Member();
				member.setId(id);
				member.setPassword(password);
				
				member = memberService.login(member);
				
				if(member == null) {
					forward(request, response, "main.do?action=loginInit");
				}
				else {
					
					HttpSession session = request.getSession(true);
					session.setAttribute("id", id);					
					request.setAttribute("name", member.getName());
					forward(request, response, "index.jsp");
				
				}
				break;
```
로그인의 경우 db에 접근하는 부분은 이전과 동일하다. 핵심 부분은 db에서 값을 가져와서 그 이후에 처리하는 부분이다. 디비에 없는 정보라면 member는 `null`을 리턴하게 된다. 해당 경우에는 forward를 통해서 다시 로그인 창으로 들어오도록 코드를 작성했다. 

로그인이 성공적으로 이뤄진 경우에는 세션을 설정하고 request에 이름을 저장해서 forwarding을 시켰다. 로그인 성공시에는 기본 페이지로 돌아가는데 이때 request attribute테이블에 저장된 name값을 읽어서 렌더링을 시킨다. 

### 로그인 뷰
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>

<%@ taglib prefix="c" uri="jakarta.tags.core" %>

<!DOCTYPE html>
	<% if (session.getAttribute("id") == null) { %>
	    <a href="main.do?action=loginInit">로그인</a>
	<% } else {%>
		<div>
			<span> ${param.name } 님 로그인 중</span> 
			<a href="main.do?action=logout">로그아웃</a>
		</div>	    
	<% } %>	
```
위는 jsp 뷰 페이지다. 로그인에 성공한 경우와 성공하지 않은 경우 다른 페이지를 보여주기 위해서 조건부 렌더링을 쓴 코드이다. 
`<% if ... %>` 태그를 활용하면 안에 자바 코드를 작성할 수 있다. 위 경우는 id라는 키값으로 세션이 설정되지 않았다면 로그인 페이지를 보여주도록 하고 그렇지 않다면 이름 정보와 로그아웃버튼을 보여준다. 

해당 페이지는 nav.jsp로 index.jsp에서 include하고 있다. 
```html
<jsp:include page="nav.jsp" >
  <jsp:param name="name" value="${name }" />
</jsp:include>
		
```

위 코드는 index.jsp에 있다. `<jsp:param name ...>`이 부분을 통해서 nav.jsp에 값을 전달할 수 있다. 

### 페이지가 바뀌는 다양한 방식
1. jsp view 페이지에서 action 패러미터를 통해 메인 서블릿으로 접근하는 방법
    ```html
      <a href="main.do?action=list">차량 목록 페이지</a>
    ```
    jsp구조에서는 다음과 같은 방식이 권장된다고 한다. 직업 자원을 요청하는 GET요청보다는 서블릿에 맵핑된 url로 요청을 보내 서블릿이 처리하도록 하는 방식이 좀 더 보안에 좋다고 한다. 왜냐하면 내부 구조를 보여주지 않기 때문이다. 

2. jsp form 태그를 통해 접근하는 방법
    ```html
    <form action="./../main.do" method="post">
      <fieldset>
        <input type="hidden" name="action" value="regist"> <br>
        <label> 차 번호 <input type="text" name="number"></label> <br>
        <label> 모델 <input type="text" name="model"></label> <br>
        <label> 가격 <input type="number" name="price"></label> <br>
        <label> 브랜드 <input type="text" name="brand"></label> <br>
        <label> 차량 크기 <select name="size">
            <option value="소형">소형</option>
            <option value="중형">중형</option>
            <option value="대형">대형</option>
        </select> 
        </label> <br> <input type="submit" value="등록"> 
        <br> 
        <a href="./../main.do?action=list">목록으로</a>
      </fieldset>
    </form>
    ```
    form태그를 활용해서도 요청이 가능하다. 

3. JSP 내부에서의 forward
    ```java
    private void forward(HttpServletRequest request, HttpServletResponse response, String path) throws ServletException, IOException {
        RequestDispatcher disp = request.getRequestDispatcher(path);
        disp.forward(request, response);
      }
    ```
  1, 2의 경우는 브라우저 url창이 바뀐다 하지만 forward를 사용하면 url이 바뀌지 않는다는 특징이 있다. 
## 느낀점
1. MVC 모델은 구현할 때 조금 귀찮은 느낌이 있었다. 하지만 분명 프로젝트가 커지면 굉장히 유용할것 같다는 느낌을 받았다.  
2. 이제 왜 html과 같은 자원 uri가 아닌데도 페이지가 렌더링이 되고 처리가되는지 알았다. 내부적으로 서블릿의 도움을 받아 동적인 처리가 가능하다는 것을 알게됐고 백엔드 개발의 매력을 느꼈다.  
3. jsp는 뭔가 리액트와 비슷한 느낌을 받았다. 물론 server side  렌더링이 되는 거지만 param을 내부로 전달하는 것부터 해서 리액트와 비슷한 느낌을 받았다. 