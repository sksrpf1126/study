# **_Servlet(서블릿)_**

- ## **_자바 진영의 웹 역사_**

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/191687582-236a0399-0f2d-47a9-8ae1-d96b97bd7506.png" width = 70%>
</p>
출처 : https://www.youtube.com/watch?v=PH8-V6ah0XQ

</br>

제일 초기에는 Web HTML 즉, 정적인 페이지만 보여주는 단계였다.  
이후, 웹에서 동적인 페이지를 만들고 싶었고, 그래서 등장한 것이 Servlet이라는 자바 기반의 기술이다. 자바 안에 HTML코드가 들어있는 구조로써, 자바를 통해 여러 데이터를 조작하여 HTML에 붙여서 화면에 보여주는게 가능하였다.

하지만 Servlet은 자바안에 HTML 코드가 들어있다 보니, 코드의 작성에 많은 어려움을 겪었고, 그래서 등장한 것이 JSP이다.  
JSP는 HTML코드 안에 JAVA코드가 들어있는 구조로, 기존의 Servlet 방식보다 개발하기가 편했지만, 시간이 지날수록 JSP코드가 복잡해지며 유지보수에 매우 좋지 않았다. 그래서 HTML코드와 JAVA코드를 분리하는 즉 MVC구조(패턴)이 등장하였다.  
MVC패턴은 간단하게 표현하자면 Model View Controller 의 약자로, 클라이언트의 요청에 따라 컨트롤러에서 데이터가 필요하면 Model을 통하여 DB에 접근하여 데이터를 가져오고 View에서 보여줄 화면을 데이터와 함께 만들어서 클라이언트 측에 제공한다. (데이터가 필요없는 화면인 경우에는 바로 View로)  
하지만 MVC에 대한 표준 없이 개발자들이 개발을 하다보니, 여러 문제가 생기게 되었고, 이러한 문제를 해결하기 위해 등장한 것이 바로 MVC 프레임워크이다.

자바의 대표적인 MVC 프레임워크는 스프링과 스프링부트가 있다.  
위와같은 기나긴 자바 웹 개발자들의 겨울을 끝내고 봄이 왔다는 의미로써, 이름을 스프링으로 지었다고 한다.(?)  
MVC패턴의 표준을 정함으로써, 여러 개발자들간의 협업이나 유지보수 측면 등 다양한 장점을 가지게 되었으며, 특히 스프링으로 넘어오면서 Servlet에 대해 개발자가 크게 신경을 안써도 되게 바뀌었다. 이는 뒤에서 설명

</br>

---

</br>

- ## **_Servlet이란?_**

  서블릿이란 동적인 웹 페이지를 만들기 위해서 사용되는 자바 기반의 웹 어플리케이션 기술이다.  
  자바 진영에서만 사용되는 기술이기 때문에 다른 언어들과는 전혀 관계가 없다.

  </br>

- ## **_서블릿 구조_**

```java
public interface Servlet {

    public void init(ServletConfig config) throws ServletException;


    public ServletConfig getServletConfig();

    public void service(ServletRequest req, ServletResponse res)
            throws ServletException, IOException;

    public String getServletInfo();

    public void destroy();
}
```

서블릿의 구조는 인터페이스로 구성이 되어 있으며, 처음에 서블릿이 만들어 질때에는 init이 호출되어 단 하나의 서블릿 객체만 만들어진다. (싱글톤 패턴)  
service 메서드를 통해 서블릿 객체를 통해 실행할 코드를 작성한다. 자세히는 Servlet을 구현하는 HttpServlet이라는 클래스를 통해 작성하여 실행한다.  
destroy 메서드는 보통 서블릿컨테이너가 종료가 될 때에 실행되며, 해당 서블릿 인스턴스를 제거한다.

</br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/191693619-9993f36a-5d84-454e-9422-6e4ce38465f5.png" width = 70%>
  </p>

출처 : https://www.youtube.com/watch?v=calGCwG_B4Y

위는 httpServlet의 간략한 구조를 나타낸다. (실제 코드는 길다)  
간단하게 보면 결국 service라는 메서드에서 클라이언트에서 보낸 http요청에서 메서드를 받아서 해당 http메서드에 따라 실행될 메서드를 구분한 것이다.

</br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/191694550-564d7a4e-a445-49d8-99a0-1d9f9301db59.png" width = 70%>
  </p>

출처 : https://webfirewood.tistory.com/38

위는 url이 /myservlet이며, http요청메서드가 get일 때 실행될 코드를 정의한 내용이다.  
 위처럼 자바코드안에 html코드가 들어가 있는 것이 jsp 이전의 servlet 기술만을 이용하여 동적인 웹 페이지를 만들때 사용하던 방식인 것이다.

</br>

- ## **_서블릿과 서블릿컨테이너_**

  지금까지 서블릿이 무엇인지, 왜 쓰는지, 어떻게 이루어져있고 어떻게 사용을 하는지를 보았다. 그럼 서블릿이라는 객체를 만들고나서는 어떻게 동작이 되는걸까?

  </br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/191691897-d1bf2df8-adf0-4f99-8cc1-f0d40251b784.png" width = 70%>
  </p>

  출처 : https://www.youtube.com/watch?v=calGCwG_B4Y

  동적인 웹 페이지를 만들기 위해서 서블릿을 사용하고, 서블릿을 관리하는 그릇을 서블릿 컨테이너라고 한다. 자바에서 대표적으로 사용하는 서블릿 컨테이너 중 하나가 Tomcat이다. 그리고 Tomcat은 WAS라고 부르기도 한다.  
  보통 앞단에 웹서버를 두어 정적인 컨텐츠를 처리하게 하고, 동적인 컨텐츠는 웹서버가 WAS로 전달하여, WAS가 동적인 컨텐츠를 처리한다. (그렇다고 웹서버가 동적인 컨텐츠를 처리 못하는 것은 아니다. IIS, ASP 등이 존재한다.)  
  그럼 톰캣, 즉 서블릿 컨테이너는 서블릿을 어떻게 관리하고 사용을 하는가?

  1. 클라이언트가 /hello와 같은 url로 http요청을 한다.
  2. 서블릿 컨테이너는 클라이언트의 요청에 따라 web.xml(서블릿 클래스의 경로 및 위치와 매핑되는 url의 정보가 기록되어 있는 파일)을 통해 요청에 맞는 서블릿을 찾는다. 이때 서블릿 객체가 존재하지 않는다면 생성을 한다.
  3. 실행할 서블릿을 찾았다면, 서블릿 컨테이너는 httpRequest, httpResponse 객체를 만들어서 해당 서블릿에 같이 전달해 준다. (service 메서드가 이를 받아서 사용)
  4. http 메서드에 맞는 doXXX 메서드를 실행하며, 클라이언트에게 httpResponse를 전달해준다.

  큰 흐름으로 위처럼 동작하여 실행하게 되는 것이다.  
  추가적으로, 하나의 요청에 하나의 서블릿이 매칭이 되는데 이때 하나의 쓰레드를 만들어서 실행하게 된다. 즉, 여러 요청이 들어오면 각각의 요청마다 쓰레드를 만들기 때문에 서블릿 컨테이너는 멀티쓰레드 방식으로 동작한다.  
  그래서 단 하나의 서블릿 인스턴스 만을 사용하기 때문에 쓰레드 동기화를 고려하여 만들어야 한다.

  </br>

---

## **_Spring에서의 Servlet?_**

지금까지 설명한 내용은 MVC 프레임워크, 즉 spring이 등장하기 전까지의 서블릿 동작과정이었다. 하지만 스프링이 등장하면서 위 servlet의 동작방식을 좀 더 효율적으로 그리고 spring **_프레임워크_** 의 표준에 동작하도록 변경하였다.

Spring 이전에는 httpServlet을 상속받는 각각의 서블릿으로 구현을 하다보니, 서블릿끼리 중복된 공통로직들을 처리할 방법이 없었다.  
하지만 DispatcherServlet이라는 단 하나의 서블릿을 맨 앞단에 둠으로써, 이를 해결하였다.

</br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/191727499-effdff9f-c394-403a-a2be-09b0236bd1b9.png" width = 70%>
  </p>
  출처 : https://yenbook.tistory.com/6

  </br>

위 이미지는 spring에서의 동작원리이다.  
DispatcherServlet은 **_FrontController 패턴 + RequestDispatcher_** 이다.  
FrontController 패턴은 클라이언트(브라우저)로부터 온 모든 요청을 다 받아서 요청한 주소에 해당하는 자원을 찾아갈 수 있도록 지원하는 것이다.  
RequestDispatcher는 FrontController에 도착한 request와 response를 기억해 둔다. 이 둘을 합쳐서 동작하는 것이 DispatcherServlet이다.

위 이미지를 토대로 Spring에서의 동적 웹페이지를 실행하기까지의 단계를 살펴보면

1번. 클라이언트(브라우저)로부터 요청이 들어온다.  
2번. DispatcherServlet은 요청된 URL을 Handler Mapping **_객체_** 에 넘기고, 해당 객체를 통해 호출해야 할 Controller 메소드 정보를 얻는다.  
3~4번. DispatcherServlet은 얻은 정보를 Handler Adapter에게 전달하고, Handler Adapter를 통해 Controller를 실행한다. (실질적으로 컨트롤러를 실행하는 것은 Handler Adapter이기 때문)
5번. Controller 객체는 비즈니스 로직을 처리하고, 그 결과를 토대로 뷰에 전달할 정보는 Model 객체에 담으며, View Name을 DispatcherServlet에게 전달한다.  
6번. DispatcherServlet은 전달받은 View Name을 View Resolver에게 전달하고, 이후 View 객체를 얻는다.  
7번. DispatcherServlet은 View 객체에게 화면 표시를 의뢰한다.  
8번. View 객체에 해당하는 뷰를 호출하고, 뷰는 Model 객체로부터 필요한 객체(정보)를 가져와 화면 표시를 한다.

</br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/191731168-2431039a-dc18-4db0-b827-cc67c45d7432.png" width = 70%>
  </p>

출처 : https://www.youtube.com/watch?v=calGCwG_B4Y  
 (3번은 View Resolver 이다.)

1번, 2번, 3번은 DispatcherServlet이 스프링 컨테이너로부터 주입받아서 자동으로 해주기 때문에 개발자는 결국 비즈니스 로직에만 집중해서 개발을 할 수 있는 환경이 마련이 된다.

왼쪽 상단은 WebApplicationContext는 DispatcherServlet이 생성이 될 때 같이 생성이 되는데, 스프링 컨테이너의 큰 틀이다.  
 Servlet WebApplicationContext와 Root WebApplicationContext로 나눠진다.

Servlet WebApplicationContext는 Controller, ViewResolver, HandlerMapping과 같은 웹 요청 처리 관련 객체들을 저장하고 관리한다.

Root WebApplicationContext는 웹 요청 처리 관련 이외의 빈들 그러니까 Services, Repositories와 관련된 객체들을 저장하고 관리한다.

Root가 먼저 만들어지고 이후에 Servlet이 만들어지기 때문에 Servlet WebApplicationContext의 객체(빈)들만이 Root WebApplicationContext 객체(빈)들에 대해 접근이 가능하다. 반대는 불가능하다.

</br>

---

## **_추가로 공부할 내용_**

- MVC 패턴 공부
- Spring 과 Spring Boot의 차이점
- 웹서버와 WAS에 대하여 + 웹서버도 동적인 컨텐츠를 처리할 수 있다에 대한 내용

</br>

---

# **_참고_**

- https://www.youtube.com/watch?v=PH8-V6ah0XQ (Servlet - Jsp - MVC - Spring 진화과정)
- https://www.youtube.com/watch?v=calGCwG_B4Y (servlet vs spring 우아한 테크)
- https://yenbook.tistory.com/6 (spring 동작 원리)
- https://velog.io/@nrudev/3.-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8-%EB%8F%99%EC%9E%91%EC%9B%90%EB%A6%AC (스프링 부트 동작 원리)
