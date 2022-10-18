# **_서블릿 -> jsp -> MVC 패턴 과정_**

해당 내용은 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술(김영한님)의 강의를 보고 정리한 내용입니다.

---

앞에서는 자바진영이 초기에 동적인 코드를 서블릿이라는 기술로 해결하는 방법을 살펴보았다.

이번에는 스프링 MVC가 등장하기 이전까지의 발전과정을 간략한 예제로 살펴볼 것이다.

발전과정은  
**_servlet -> jsp -> MVC 패턴 -> 스프링 MVC_** 정도이다.

</br>

---

## **_servlet_**

예제는 간단하게, 회원가입 폼과 회원가입 이후 가입한 정보를 확인하는 save 페이지가 있다. (회원리스트 페이지도 있지만 생략)

```java
@WebServlet(name = "memberFormServlet", urlPatterns = "/servlet/members/new-form")
public class MemberFormServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");

        PrintWriter w = response.getWriter();
        w.write("<!DOCTYPE html>\n" +
                "<html>\n" +
                "<head>\n" +
                " <meta charset=\"UTF-8\">\n" +
                " <title>Title</title>\n" +
                "</head>\n" +
                "<body>\n" +
                "<form action=\"/servlet/members/save\" method=\"post\">\n" +
                " username: <input type=\"text\" name=\"username\" />\n" +
                " age: <input type=\"text\" name=\"age\" />\n" +
                " <button type=\"submit\">전송</button>\n" +
                "</form>\n" +
                "</body>\n" +
                "</html>\n");
    }
}
```

위는 회원 가입 폼 서블릿 예제이다.

</br>

```java
@WebServlet(name = "memberSaveServlet", urlPatterns = "/servlet/members/save")
public class MemberSaveServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");
        PrintWriter w = response.getWriter();

        w.write("<html>\n" +
                "<head>\n" +
                " <meta charset=\"UTF-8\">\n" +
                "</head>\n" +
                "<body>\n" +
                "성공\n" +
                "<ul>\n" +
                " <li>id="+member.getId()+"</li>\n" +
                " <li>username="+member.getUsername()+"</li>\n" +
                " <li>age="+member.getAge()+"</li>\n" +
                "</ul>\n" +
                "<a href=\"/index.html\">메인</a>\n" +
                "</body>\n" +
                "</html>");
    }
}
```

위는 회원가입 이후의 save 페이지이다.

만들면서 느낀 부분이 바로 html 작성 부분이다.

정말 매우 단순한 화면만 보여주기에 html 코드도 정말 적지만, 그것만으로도 매우 작성하기 불편하고, html 코드의 에러 또한 단순히 문자열로 정의하기에 찾기도 매우 힘들다.

위처럼 java코드 안에 html 코드를 작성하는 방법이 아닌 html 코드에서 필요한 부분들만 java코드를 호출해서 사용하는 것은 더 깔끔하지 않을까? 해서 나온 방법이 바로 jsp이다.

```
참고로 JSP는 성능과 기능면에서 다른 템플릿 엔진에 밀려서 현재는 사장되어 가는 추세이다.

스프링에서도 jsp는 추천하지 않고, Thymeleaf를 밀고 있다.
```

</br>

---

## **_JSP_**

```Java
 <%@ page contentType="text/html;charset=UTF-8" language="java" %>
 <html>
 <head>
    <title>Title</title>
 </head>
 <body>
     <form action="/jsp/members/save.jsp" method="post">
     username: <input type="text" name="username" />
     age: <input type="text" name="age" />
     <button type="submit">전송</button>
     </form>
 </body>
 </html>
```

JSP방식의 회원가입 폼 페이지이다.

</br>

```java
<%@ page import="hello.servlet.domain.member.MemberRepository" %>
<%@ page import="hello.servlet.domain.member.Member" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    //request, response 사용 가능
MemberRepository memberRepository = MemberRepository.getInstance();
System.out.println("save.jsp");
String username = request.getParameter("username");
int age = Integer.parseInt(request.getParameter("age"));
Member member = new Member(username, age);
System.out.println("member = " + member);
memberRepository.save(member);
%>
<html>
<head>
<meta charset="UTF-8">
</head>
<body>
성공
<ul>
<li>id=<%=member.getId()%></li>
<li>username=<%=member.getUsername()%></li>
<li>age=<%=member.getAge()%></li>
</ul>
<a href="/index.html">메인</a>
</body>
</html>
```

JSP 방식의 save 페이지이다.

서블릿과 반대되게 html 코드에서 java코드를 사용하는 방식이라 HTML 부분에 있어 좀 더 깔끔하게 코드를 작성할 수 있다.

중간마다 필요한 부분에서만 java코드를 작성하면 되기에 유지보수 방면에서도 더 뛰어나 보인다.

하지만 위 방식 또한 여러 문제가 발생한다.

위 코드를 보면 데이터를 저장하고 조회하는 java 코드 즉, 비즈니스 로직 부분들과 UI부분 즉, HTML 코드가 하나의 파일에 모두 작성이 되어 있다.

하나의 파일이 많은 역할을 담당하는 것이다.

만약 몇천줄이 되는 jsp 파일 하나가 있다면, 유지보수에만 엄청난 힘을 쏟아야 할 것이다.

그러면 위의 비즈니스 로직과 HTML 부분을 분리하면 어떨까? 에서 나온 것이 MVC 패턴인 것이다.

</br>

---

## **_MVC 패턴_**

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/196379683-54d31e7a-74a5-450d-96c9-44ec5065fbcd.png" width = 70%>
</p>

위는 MVC패턴을 이미지로 표현한 것이다.

클라이언트의 요청이 오면 컨트롤러가 받아서 필요한 비즈니스 로직단을 서비스나 리포지토리 부분에서 처리를 하고 이후 처리한 데이터는 컨트롤러가 Model에 담는다.

이후 뷰 부분에서 화면에 필요한 동적인 데이터들은 Model에서 가져와서 사용한다.

```java
@WebServlet(name = "mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/new-form")
public class MvcMemberFormServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";

        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
<meta charset="UTF-8">
<title>Title</title>
</head>
<body>
<!-- 상대경로 사용, [현재 URL이 속한 계층 경로 + /save] -->
<form action="save" method="post">
username: <input type="text" name="username" />
age: <input type="text" name="age" />
<button type="submit">전송</button>
</form>
</body>
</html>
```

위 2개는 회원가입 폼을 처리하는 부분이다.

</br>

```java
@WebServlet(name = "mvcMemberSaveServlet", urlPatterns = "/servlet-mvc/members/save")
public class MvcMemberSaveServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관한다.
        request.setAttribute("member", member);
        String viewPath = "/WEB-INF/views/save-result.jsp";

        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
<meta charset="UTF-8">
</head>
<body>
성공
<ul>
<li>id=${member.id}</li>
<li>username=${member.username}</li>
<li>age=${member.age}</li>
</ul>
<a href="/index.html">메인</a>
</body>
</html>
```

위 부분들이 save 페이지를 처리하는 부분이다.

코드를 보면 비즈니스 로직과 HTML(UI) 부분이 분리가 된 것을 확인할 수 있으며, 필요한 데이터들만 가져다가 사용한다.

하지만 이러한 MVC패턴도 한계가 있다.

우선 공통 처리가 어렵다는 것인데, 위 부분에서 포워드 부분이 중복이며, viewPath 또한 앞의 고정된 부분이 중복이다.

프로젝트가 커지면 커질수록 공통로직이 증가할 것인데, 위 부분들을 공통메서드로 정의한들 결국 모든 컨트롤러 부분에서 해당 메서드를 호출하는 것은 마찬가지이다.

다음으로 request, response객체를 잘 사용하지 않게 되고, 테스트 코드작성에도 어려움이 있다는 것이다.

위 공통 문제를 해결하기 위해서는 각각의 컨트롤러들이 실행하기 이전에 앞에서 공통 로직들을 처리하는 **_수문장_** 역할을 하는 부분이 필요하다.

그렇기에 나온 방법이 **_Front Controller 패턴_** 이다.

위 패턴이 적용된 것이 바로 스프링 MVC이다.
