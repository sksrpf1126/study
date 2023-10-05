# **_Spring MVC_**

해당 내용은 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술(김영한님)의 강의를 보고 정리한 내용입니다.

</br>

이전에는 직접 spring MVC 프레임워크과 비슷하게 MVC 프레임워크를 만들어 봤다.

이번에는 spring MVC 프레임워크의 구조와 만들어본 MVC 프레임워크를 비교하며, spring MVC 동작 방식을 살펴보자.

</br>

## **_비교_**

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/196867441-854a519f-2103-4cf1-8b02-36258aa2ab33.png" width = 70%>
</p>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/196895267-5717e6ed-dd13-48b9-8437-c33c678bb1b4.png" width = 70%>
</p>

위에 이미지가 이전의 V5버전이며, 아래가 spring MVC 구조를 표현한 것이다.

**_직접 만든 프레임워크 스프링 MVC 비교_**

FrontController -> DispatcherServlet  
handlerMappingMap -> HandlerMapping  
MyHandlerAdapter -> HandlerAdapter  
ModelView -> ModelAndView  
viewResolver -> ViewResolver  
MyView -> View

구조는 거의 같다고 보면 된다.

동작방식도 큰 틀에서 거의 동일하지만, 당연히 spring MVC가 매우 엄청 상위호환을 가진다.

그리고 이러한 부분들은 전부 공부할 수가 없을 정도로 방대하다.

주의 깊게 봐야할 부분은 기존에는 핸들러 매핑의 경우에 Map에 담아서 클라이언트의 요청URI를 기준으로 반환했고, 핸들러 어댑터 또한 직접 V3와 V4를 만들었다.

그러면 spring MVC는 핸들러 부분을 어떻게 처리하는가?

### **_HandlerMapping_**

```
0순위 RequestMappingHandlerMapping : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
1순위 BeanNameUrlHandlerMapping : 스프링 빈의 이름으로 핸들러를 찾는다.
```

실제로는 더 많다.

위 HandlerMapping들을 순서대로 실행하며, 핸들러를 찾는다.

요청URI를 정의해 둔 컨트롤러가 애노테이션인지, 아니면 스프링 빈의 이름으로 정의해 두고 사용하는지에 따라 핸들러를 반환한다.

RequestMappingHandlerMapping은 스프링 빈 중에서 클래스 레벨에 @RequestMapping 또는 @Controller가 붙어 있는지를 보고 매핑 정보를 인식한다.

### **_HandlerAdapter_**

```
0 = RequestMappingHandlerAdapter : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
1 = HttpRequestHandlerAdapter : HttpRequestHandler 처리
2 = SimpleControllerHandlerAdapter : Controller 인터페이스(애노테이션X, 과거에 사용) 처리
```

실제로는 더 많다.
어떤 기반으로 핸들러를 반환했는지에 따라 어댑터 또한 스프링이 선택해서 실행시킨다.

정리하면, 이미 스프링은 지금까지 발전해오면서 대부분의 기능들을 넣었기 때문에 이전의 V5처럼 핸들러 어댑터를 만드는 일은 거의 없다고 보면 된다.

</br>

---

## **_이전 코드를 스프링 MVC로_**

```java
@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @GetMapping("/new-form")
    public String newForm(){
        return "new-form";
    }

    @PostMapping("/save")
    public String save(@RequestParam String username, @RequestParam int age, Model model) {

        Member member = new Member(username, age);
        memberRepository.save(member);

        model.addAttribute("member", member);
        return "save-result";
    }

    @GetMapping()
    public String members(Model model) {
        List<Member> members = memberRepository.findAll();

        model.addAttribute("members",members);
        return "members";
    }

}
```

@Controller 애노테이션을 보고 dispatcherServlet은 핸들러 매핑의 정보를 얻어온다.

이후, 핸들러를 통해 해당 핸들러와 호환되는(사용할 수 있는) 어댑터를 가져오고, 이를 통해 컨트롤러를 실행시키는 것이다.

메서드에 붙어있는 @RequestParam, @RequestBody, Model 객체 등을 보고 스프링이 이러한 부분들도 알아서 맞게 제공해준다.
