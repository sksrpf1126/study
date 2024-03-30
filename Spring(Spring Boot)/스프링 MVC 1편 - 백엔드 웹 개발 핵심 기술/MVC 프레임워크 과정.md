# **_MVC 프레임워크 과정_**

해당 내용은 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술(김영한님)의 강의를 보고 정리한 내용입니다.

</br>

---

https://github.com/sksrpf1126/study/blob/main/Spring(Spring%20Boot)/%EC%8A%A4%ED%94%84%EB%A7%81%20MVC%201%ED%8E%B8%20-%20%EB%B0%B1%EC%97%94%EB%93%9C%20%EC%9B%B9%20%EA%B0%9C%EB%B0%9C%20%ED%95%B5%EC%8B%AC%20%EA%B8%B0%EC%88%A0/%EC%84%9C%EB%B8%94%EB%A6%BF%2Cjsp%2CMVC%20%EA%B3%BC%EC%A0%95.md

위에서 서블릿 -> JSP -> MVC 프레임워크의 발전 과정을 예시로 살펴봤다면 이번에는 MVC 프레임워크를 스프링 MVC와 매우 유사하게 만들어 볼 것이다.(발전과정의 최종 버전이 현재 스프링 MVC가 사용하는 방법과 매우 유사)

버전은 V1 ~ V5까지 존재한다.

</br>

---

## **_MVC 프레임워크 V1_**

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/196661129-aaeeb80f-7b4b-45d1-a2ed-e9da9c420fa4.png" width = 70%>
</p>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/196661149-34f773cc-3822-4374-a1f9-e622329eaf9a.png" width = 70%>
</p>

front controller를 도입하기 전에는 컨트롤러에 존재하는 공통 로직들을 처리할 수가 없었다. 각 컨트롤러마다 다른 서블릿이 동작이 되기 때문이다.

하지만 front controller 서블릿 단 하나만 맨 앞에 둬서 모든 클라이언트의 요청을 받도록 하고, 해당 서블릿에서 공통 로직을 처리하게끔 하면 된다.

스프링 MVC 또한 바로 이 front controller 패턴을 사용한다. (Dispatcher Servlet)

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/196662320-44840a0b-d1af-426e-b750-652669f738ca.png" width = 70%>
</p>

위는 v1의 구조를 간략히 표현한 것이다.

Front Controller 서블릿이 맨 앞단에서 클라이언트의 요청을 받고, 클라이언트가 요청한 URI정보를 토대로 매핑 정보에서 어떤 컨트롤러를 호출할지에 대한 정보를 가져온다.

이후 컨트롤러를 호출하고, 컨트롤러에서 JSP를 forward한다.

그럼 간단하게 위 패턴을 도입해보자.

</br>

## **_front controller v1_**

```java
///front-controller/v1/* -> v1 이하의 url들은 해당 서블릿이 전부 받음
@WebServlet(name = "frontControllerServletV1", urlPatterns = "/front-controller/v1/*")
public class FrontControllerServletV1 extends HttpServlet {

    private Map<String, ControllerV1> controllerMap = new HashMap<>();

    public FrontControllerServletV1() {
        controllerMap.put("/front-controller/v1/members/new-form", new MemberFormControllerV1());
        controllerMap.put("/front-controller/v1/members/save", new MemberSaveControllerV1());
        controllerMap.put("/front-controller/v1/members", new MemberListControllerV1());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String requestURI = request.getRequestURI();

        ControllerV1 controller = controllerMap.get(requestURI);
        if(controller == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND); //404에러 반환
            return;
        }

        controller.process(request, response);
    }
}
```

매핑 정보를 가지고 있는 controllerMap에서 클라이언트가 요청한 URI를 토대로 호출할 컨트롤러를 찾아서 실행한다.

</br>

## **_form controller v1_**

```java
public class MemberFormControllerV1 implements ControllerV1 {
    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";

        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

</br>

## **_save controller v1_**

```java
public class MemberSaveControllerV1 implements ControllerV1 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
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

</br>

## **_member list controller v1_**

```java
public class MemberListControllerV1 implements ControllerV1 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        List<Member> members = memberRepository.findAll();

        request.setAttribute("members", members);

        String viewPath = "/WEB-INF/views/members.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

컨틀롤러에서 내부적으로 비즈니스 로직을 처리하며, viewName(viewPath)를 통해서 JSP를 forward한다.

v1 버전에서는 Front Controller 패턴을 사용하긴 했지만 컨트롤러 코드를 보면, 비즈니스 로직 외에 viewName을 통해서 JSP를 forward하는 **_공통부분_** 들이 중복이 된다.

</br>

---

## **_MVC 프레임워크 V2_**

v2에서는 컨트롤러마다 중복되는 JSP forward 부분을 해결해보자.

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/196867423-74d794bb-5708-4a30-843e-e4b0230d58aa.png" width = 70%>
</p>

V2 버전의 구조를 보여주는 이미지이다.

v1 버전에서 Controller에서 JSP를 forward를 해주었다면, 해당 버전에서는 Controller는 MyView 객체를 다시 Front Controller에 반환을 하고, JSP forward 처리 또한 Front Controller에서 처리한다.

## **_MyView 객체_**

```java
public class MyView {
    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

우선 view를 처리해줄 객체를 만든다. (v3~v5에서도 사용 예정)

viewPath를 받으며, render 메서드를 통해서 JSP forward를 할 수 있다.

</br>

## **_front controller v2_**

```java
@WebServlet(name = "frontControllerServletV2", urlPatterns = "/front-controller/v2/*")
public class FrontControllerServletV2 extends HttpServlet {

    private Map<String, ControllerV2> controllerMap = new HashMap<>();

    public FrontControllerServletV2() {
        controllerMap.put("/front-controller/v2/members/new-form", new MemberFormControllerV2());
        controllerMap.put("/front-controller/v2/members/save", new MemberSaveControllerV2());
        controllerMap.put("/front-controller/v2/members", new MemberListControllerV2());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String requestURI = request.getRequestURI();

        ControllerV2 controller = controllerMap.get(requestURI);
        if(controller == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND); //404에러 반환
            return;
        }

        MyView view = controller.process(request, response);
        view.render(request, response);
    }
}
```

v2 버전의 front controller servlet 코드이다.

달리진 부분은 컨트롤러를 호출만 하고 끝나는 것이 아니라 호출 후에 MyView 객체를 받아서 참조변수 view에 담는다.

이후 view 참조변수를 통해 render 메서드를 호출하여 JSP forward를 처리한다. (controller에서 공통으로 하는 부분을 front controller servlet에서 처리)

</br>

## **_form controller v2_**

```java
public class MemberFormControllerV2 implements ControllerV2 {
    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        return new MyView("/WEB-INF/views/new-form.jsp");
    }
}
```

jsp forward 부분만 존재하던 form controller는 이 부분마저 front controller에서 처리를 해주기 때문에 viewPath를 통해 MyView 객체를 만들어서 반환만 해주게끔 변경되었다.

</br>

## **_save controller v2_**

```java
public class MemberSaveControllerV2 implements ControllerV2 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관한다.
        request.setAttribute("member", member);

        return new MyView("/WEB-INF/views/save-result.jsp");
    }
}
```

</br>

## **_member list controller v2_**

```java
public class MemberListControllerV2 implements ControllerV2 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        List<Member> members = memberRepository.findAll();
        request.setAttribute("members", members);
        return new MyView("/WEB-INF/views/members.jsp");
    }
}
```

save controller 와 member list controller 또한 view를 처리하지 않고, MyView 객체만 반환하게끔 변경되었다.

이를 통해 v1-> v2로 변경되면서, 공통인 로직은 front controller servlet에서 처리하게 변경되었고, 컨트롤러에서는 좀 더 비즈니스 로직에 집중할 수 있도록 개선 되었다.

하지만 아직도 부족한 부분이 존재한다.

1. 컨트롤러가 서블릿에 종속적이다.  
   -> 컨트롤러에서 HttpServletRequest, HttpServletResponse를 받게끔 구현이 되어있다.  
   즉, 서블릿에 종속적이라는 것이다. 이러한 부분들은 서블릿에서 처리해야지 컨트롤러에서 처리할 수 있는 여지를 주면 안된다. Model 객체를 활용하여 이를 해결해보자.

2. 뷰 이름 중복  
   -> /WEB-INF/views 와 뒤에 .jsp 부분은 실제 물리 위치에 고정적인 이름으로, 중복이 발생한다.  
   논리적인 이름(컨트롤러를 구분하는 이름)만을 활용해서도 똑같이 구현되게끔 개선을 해보자.

</br>

---

## **_MVC 프레임워크 V3_**

V2버전에서 컨트롤러가 서블릿에 종속적인점, 뷰 이름이 중복인점을 해결해보자.

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/196867430-9f6f104e-47a2-440a-bbf2-8b3983bf5f7f.png" width = 70%>

V2버전에서 Controller가 MyView 객체를 반환해주고 Front Controller에서는 해당 객체를 통해 JSP를 forward를 해준다.

V3버전에서는 Controller는 ModelView객체를 반환하고, Front Controller는 ModelView 객체에 담겨있는 viewName을 통해서 viewResolver 메서드(간단하게 메서드로 구현한 것)를 호출하여 MyView 객체를 얻어온다.

MyView 객체를 통해서 이후에는 jsp forward를 해준다.

코드로 살펴보자.

</br>

## **_ModelView 객체_**

```java
@Getter @Setter
public class ModelView {
    private String viewName;
    private Map<String, Object> model = new HashMap<>();

    public ModelView(String viewName){
        this.viewName = viewName;
    }
}
```

view의 이름을 저장할 viewName과 컨트롤러의 HttpServletRequest 와 Response를 사용하지 않기 위한 model 객체(Map)가 정의되어 있다.

</br>

## **_front controller V3_**

```java
@WebServlet(name = "frontControllerServletV3", urlPatterns = "/front-controller/v3/*")
public class FrontControllerServletV3 extends HttpServlet {

    private Map<String, ControllerV3> controllerMap = new HashMap<>();

    public FrontControllerServletV3() {
        controllerMap.put("/front-controller/v3/members/new-form", new MemberFormControllerV3());
        controllerMap.put("/front-controller/v3/members/save", new MemberSaveControllerV3());
        controllerMap.put("/front-controller/v3/members", new MemberListControllerV3());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String requestURI = request.getRequestURI();

        ControllerV3 controller = controllerMap.get(requestURI);
        if(controller == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND); //404에러 반환
            return;
        }

        //paramMap

        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);

        String viewName = mv.getViewName();// 논리이름 new-form

        MyView view = viewResolver(viewName);

        view.render(mv.getModel(),request, response);
    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();

        request.getParameterNames().asIterator().forEachRemaining(
                paramName -> paramMap.put(paramName, request.getParameter(paramName))
        );
        return paramMap;
    }
}
```

createParamMap()를 통해 클라이언트가 요청을 보낸 데이터들을 모두 가져와서 paramMap에 저장한다.

이후 컨트롤러를 호출할 때 paramMap을 전달한다.

다음으로 컨트롤러에서 반환한 ModelView 객체에서 viewName을 가져와서 viewResolver()를 통해서 MyView 객체를 가져온다.

viewResolver()에서는 viewName만을 통해서 MyView 객체를 만들어주어서 기존의 viewName 중복을 해결하였다.

이후 view 객체의 render 메서드에서 기존의 request, response 객체 뿐만 아니라 ModelView 객체에 존재하는 model 객체 또한 넘겨준다.

</br>

## **_MyView 객체 render() 추가_**

```java
    public void render(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        modelToRequestAttribute(model, request);
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }

    private void modelToRequestAttribute(Map<String, Object> model, HttpServletRequest request) {
        model.forEach((key, value) -> request.setAttribute(key,value));
    }
```

MyView 객체에 추가된 메서드이다.

render 메서드가 하나 더 정의되어 있다.(Overloading)  
해당 메서드에서는 model에 담겨진 정보들을 request에 담아서 jsp를 forward하는 기능이다.

이러한 로직을 통해서 컨트롤러가 서블릿에 종속적인 문제를 해결하였다.

</br>

## **_form controller V3_**

```java
public class MemberFormControllerV3 implements ControllerV3 {
    @Override
    public ModelView process(Map<String, String> paramMap) {
        return new ModelView("new-form");
    }
}
```

</br>

## **_save controller v3_**

```java
public class MemberSaveControllerV3 implements ControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public ModelView process(Map<String, String> paramMap) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelView mv = new ModelView("save-result");
        mv.getModel().put("member", member);
        return mv;
    }
}
```

</br>

## **_member list controller V3_**

```java
public class MemberListControllerV3 implements ControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public ModelView process(Map<String, String> paramMap) {
        List<Member> members = memberRepository.findAll();
        ModelView mv = new ModelView("members");
        mv.getModel().put("members",members);

        return mv;
    }
}
```

클라이언트가 요청할 때 넘겨준 데이터들은 paramMap을 통해 controller에 전달하여서 비즈니스 로직을 처리한다.

그리고 만약에 응답할 때 클라이언트한테 제공해줘야 할 데이터가 존재한다면 컨트롤러는 그저 model 객체에 데이터를 담아 ModelView 객체만을 반환하면 된다.

</br>

---

## **_MVC 프레임워크 V4_**

V3도 충분히 잘 설계된 MVC 프레임워크 이지만 모든 컨트롤러가 modelView 객체를 반환하는데, 이를 좀 더 좋은 방식으로 개선할 수 있지 않을까?

V4버전은 이를 개선하는 방식이다.

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/196867431-e16a857e-888c-41c3-91ca-5657f91776fa.png" width = 70%>

V4에서 달라지는 부분은 Front Controller와 Controller 간의 모양이다.

V3에서는 paramMap을 전달하고, ModelView 객체를 받았지만, 이번에는 paramMap과 model을 전달하고 String 타입의 viewName만을 받아서 처리한다.

</br>

## **_front Controller V4_**

```java
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String requestURI = request.getRequestURI();

        ControllerV4 controller = controllerMap.get(requestURI);
        if(controller == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND); //404에러 반환
            return;
        }

        //paramMap

        Map<String, String> paramMap = createParamMap(request);
        Map<String, Object> model = new HashMap<>();
        String viewName = controller.process(paramMap, model);

        MyView view = viewResolver(viewName);

        view.render(model,request, response);
    }
```

V4에서는 ModelView 객체를 사용하지 않는다.

Front Controller에서 만든 Model 객체를 전달하면, 컨트롤러에서 비즈니스 로직을 처리한 후 Model 객체에 데이터를 담으면 이를 render()에 넘겨줄 뿐이다.

</br>

## **_member list controller V4_**

```java
public class MemberSaveControllerV4 implements ControllerV4 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public String process(Map<String, String> paramMap, Map<String, Object> model) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        model.put("member", member);
        return "save-result";
    }
}
```

하나의 컨트롤러만 봐도 좀 더 개선된 모양인 것을 확인할 수 있다.

그저 paramMap과 model은 필요에 맞게 사용하면 되고, jsp forward를 할 viewName만 반환하면 된다.

</br>

---

## **_MVC 프레임워크 V5_**

최종 버전이다.

V1 ~ V4에서는 자기 자신의 버전만 사용이 가능하다. 하지만 V3와 V4중에 선택해서 사용하고 싶은 경우에 이를 해결할 방법이 존재하지 않는다.

V5에서는 어댑터 패턴을 도입하여 이를 해결한다.

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/196867441-854a519f-2103-4cf1-8b02-36258aa2ab33.png" width = 70%>

추가된 부분은 핸들러 어댑터 목록과 핸들러 어댑터 부분이다.

코드로 바뀐 부분을 살펴보자.

</br>

## **_MyHandlerAdapter Interface_**

```java
public interface MyHandlerAdapter {

    boolean supports(Object handler);

    ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
}

```

우선 어댑터를 인터페이스로 정의해 둔다. 이후 다형성을 통해서 필요한 버전을 구현시켜서 사용하면 된다.

현재는 V3와 V4 2개만을 구현시킨다.

</br>

## **_V3 HandlerAdapter_**

```java
public class ControllerV3HandlerAdapter implements MyHandlerAdapter {
    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV3);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        ControllerV3 controller = (ControllerV3) handler;

        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);

        return mv;
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();

        request.getParameterNames().asIterator().forEachRemaining(
                paramName -> paramMap.put(paramName, request.getParameter(paramName))
        );
        return paramMap;
    }
}
```

supports()에서는 넘어온 handler 객체, 그러니까 클라이언트 측에서 요청한 컨트롤러가 V3인지 확인하는 용도이다.

handle()에서는 request, response, 그리고 handler 객체를 받는다. 이후 handler 객체 타입이 Object이므로 해당 버전에 맞게 형변환을 거친다.

그리고 V4버전까지는 createParamMap을 front Controller 단에서 처리 했지만, V5에서는 각 버전에 맞는 handle()에서 처리를 하게 한다.

그리고 중요한 것은 어떠한 버전이든지 ModelView 객체를 반환하게끔 한다. V4에서는 viewName만을 반환하는데 이럴 경우에는 ModelView 객체를 핸들러 어댑터에서 만들어서라도 반환해야 한다. (반환 타입을 맞추기 위해)

반환 타입을 맞춰야 어떠한 버전이든 front controller단에서 같은 로직으로 처리할 수 있기 때문(?)

</br>

## **_V4 handlerAdapter_**

```java
public class ControllerV4HandlerAdapter implements MyHandlerAdapter {
    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV4);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        ControllerV4 controller = (ControllerV4) handler;

        Map<String, String> paramMap = createParamMap(request);
        Map<String, Object> model = new HashMap<>();

        String viewName = controller.process(paramMap, model);

        ModelView mv = new ModelView(viewName);
        mv.setModel(model);

        return mv;

    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();

        request.getParameterNames().asIterator().forEachRemaining(
                paramName -> paramMap.put(paramName, request.getParameter(paramName))
        );
        return paramMap;
    }
}
```

V3와 거의 유사하다.

반환타입을 맞추기 위해 viewName을 통해 ModelView 객체를 생성하여 반환할 뿐이다.

</br>

## **_front controller V5_**

```java
@WebServlet(name = "frontControllerServletV5", urlPatterns = "/front-controller/v5/*")
public class FrontControllerServletV5 extends HttpServlet {

    private final Map<String, Object> handlerMappingMap = new HashMap<>();
    private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();

    public FrontControllerServletV5() {
        initHandlerMappingMap();
        initHandlerAdapters();
    }

    private void initHandlerAdapters() {
        handlerAdapters.add(new ControllerV3HandlerAdapter());
        handlerAdapters.add(new ControllerV4HandlerAdapter());
    }

    private void initHandlerMappingMap() {
        handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new MemberFormControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members/save", new MemberSaveControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members", new MemberListControllerV3());

        //V4 추가
        handlerMappingMap.put("/front-controller/v5/v4/members/new-form", new MemberFormControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members/save", new MemberSaveControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members", new MemberListControllerV4());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Object handler = getHandler(request);

        if(handler == null){
            response.setStatus(HttpServletResponse.SC_NOT_FOUND); //404에러 반환
            return;
        }

        MyHandlerAdapter adapter = getHandlerAdapter(handler);

        ModelView mv = adapter.handle(request, response, handler);

        String viewName = mv.getViewName();// 논리이름 new-form
        MyView view = viewResolver(viewName);

        view.render(mv.getModel(),request, response);
    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }

    private MyHandlerAdapter getHandlerAdapter(Object handler) {
        for (MyHandlerAdapter adapter : handlerAdapters) {
            if(adapter.supports(handler)){
                return adapter;
            }
        }
        throw new IllegalArgumentException("handler adapter를 찾을 수 없습니다. handler = " + handler);
    }

    private Object getHandler(HttpServletRequest request) {
        String requestURI = request.getRequestURI();
        return handlerMappingMap.get(requestURI);
    }
}
```

생성자를 보면 v3와 v4의 버전의 매핑 정보를 전부 담고 있다.

이후 getHandler()를 통해 클라이언트가 요청한 URI를 통해 매핑정보를 얻어오고 이를 모든 클래스의 부모인 Object 타입으로 저장한다.

이후 getHandlerAdapter()에 handler 객체를 전달한다.  
해당 메서드에서는 handler 객체가 V3인지 V4인지를 판별하여 버전에 맞는 handlerAdapter를 반환한다.

다음으로 해당 객체의 handle 메서드를 통해 컨트롤러에 접근하여 처리 후 ModelView 객체를 가져온다.

이후 동일하게 ModelView객체를 통해 jsp forward를 처리한다.

</br>

---

## **_정리_**

지금까지 정리한 내용은 스프링 MVC 프레임워크의 핵심 코드의 축약 버전이다.

결국 위 V5방식이 스프링 MVC 프레임워크의 동작방식과 매우 유사하다는 것을 알 수 있다.

다음 내용은 현재 개발한 V5버전과 스프링 MVC를 비교하면서 스프링 MVC를 정리할 것이다.
