# **_Cookie와 Session_**

해당 내용은 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술(김영한님)의 강의를 보고 정리한 내용입니다.

---

HTTP는 클라이언트와의 상태를 유지하지 않는다.(stateless)

그러면 클라이언트가 로그인을 했을 때 로그인을 유지를 할려면 어떻게 해야 하는가?

이에 대해 여러가지 방법이 존재한다. Cookie와 Session 방법으로도 가능하며, Spring security를 활용한 방법, JWT(JSON Web Token) 방법도 존재한다.

해당 내용에서는 Cookie와 Session의 방식으로 로그인을 유지하고 로그아웃도 하는 방법을 정리한다.

</br>

---

## **_Cookie_**

Cookie란 클라이언트 측의 저장소(메모리 공간)을 활용하여 데이터를 저장하는데, 저장된 데이터를 쿠키라고 표현한다.

클라이언트는 보통 300개의 쿠키를 저장할 수 있으며, 하나의 쿠키에 4KB까지 저장할 수 있다.

이를 활용하여 클라이언트가 아이디와 비밀번호를 입력하여 서버에 요청하면, 서버는 해당 정보로 조회되는 회원이 있는지 판별하고, 존재한다면 응답(response)할 때 쿠키를 전달하여 클라이언트 측에 저장하도록 한다.

이후에 클라이언트는 요청(request)마다 쿠키 정보 또한 같이 전달하고, 서버는 지속적으로 쿠키 정보를 활용하여 로그인 여부를 판별하여 상태를 유지한다.

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/198869076-c650eb41-13cd-4329-a2dc-7e18447d4d48.png" width = 70%>
</p>

위 그림의 순서는

1. 클라이언트가 POST로 loginId와 password 데이터와 함께 로그인을 요청한다.

2. 서버는 회원 저장소에서 클라이언트가 전달한 정보를 통해 가입된 회원인지 판별한다.

3. 가입된 회원인 경우에는 로그인을 성공시키며, 응답할 때에 Cookie 정보를 전달하여 클라이언트의 쿠키 저장소에 저장시키도록 한다.

위 순서를 코드로 보자.

```java
    @PostMapping("/login")
    public String login(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletResponse response){

        //클라이언트가 보낸 데이터에 대해 에러발생 시 로그인페이지로
        if(bindingResult.hasErrors()){
            return "login/loginForm";
        }

        //회원 저장소를 통해 가입된 회원이면 가입된 회원의 정보를 반환
        Member loginMember = loginService.login(form.getLoginId(), form.getPassword());

        //가입된 회원이 아닌 경우 에러를 전달하여 보여주고 로그인페이지로
        if(loginMember == null){
            bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");//reject == globalError
            return "login/loginForm";
        }

        //쿠키에 시간정보를 주지 않으면 세션 쿠키로(브라우저 종료시에 쿠키 제거)
        Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
        response.addCookie(idCookie);

        return "redirect:/";
    }
```

외 로직을 통해 정상적으로 로그인이 되었을 경우에

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/198869343-c51f4eb9-614d-403b-b7d2-0eb4d3946a91.png" width = 70%>
</p>

응답 헤더에 Set-Cookie에 로그인한 member의 Id값을 전달한다.  
그러면 아래와 같이 클라이언트의 쿠키 저장소에 해당 데이터가 저장이 된다.

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/198869351-41436743-7b0a-44e6-a59a-ca800738e01c.png" width = 70%>
</p>

그리고 나서 "redirect:/" 에 의해서 메인페이지로 redirect를 시킨다.

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/198869512-886b2134-16df-40f1-8d3c-7584ba780db9.png" width = 70%>
</p>

그러면 위와 같이 클라이언트는 GET으로 요청을 보내고, 보낼 때에 쿠키 저장소에 저장된 쿠키를 함께 보낸다.

서버는 해당 쿠키 데이터를 보고, 해당 클라이언트가 로그인을 했는지를 판별한다.

```java
    @GetMapping("/")
    public String homeLogin(@CookieValue(name = "memberId", required = false) Long memberId, Model model){

        //쿠키에 memberId(로그인 판별 쿠키)가 존재하지 않는다면
        //로그인 전의 홈페이지 화면을 전달
        if(memberId == null){
            return "home";
        }

        //쿠키에 memberId를 통해 회원 저장소에 존재하는 회원인지 판별
        Member loginMember = memberRepository.findById(memberId);

        //존재하지 않는 회원인 경우에는 로그인 전의 홈페이지 화면 전달
        if(loginMember == null){
            return "home";
        }

        //존재하는 회원인 경우에는 로그인 이후의 페이지를 전달하여 회원의 이름을 화면에 보여줌
        model.addAttribute("member", loginMember);
        return "loginHome";
    }
```

HttpServletRequest를 통해 Cookie 데이터를 가져올 수 있지만, 스프링은 @CookieValue 애노테이션을 통해 쉽게 가져오는 방식을 제공해 준다.

위 Cookie 방식은 로그인 이후에는 브라우저 종료 전까지 매 요청마다 쿠키도 함께 전달하기 때문에 로그인을 항시 유지할 수 있다. (쿠키에 도메인, PATH 경로 정보가 있어, 다른 도메인의 쿠키와 구분하여 사용이 된다)

</br>

**_로그아웃_**

```java
    @PostMapping("/logout")
    public String logout(HttpServletResponse response){
        Cookie cookie = new Cookie("memberId", null);
        cookie.setMaxAge(0);
        response.addCookie(cookie);

        return "redirect:/";
    }
```

로그아웃의 경우에는 위와 같이 덮어씌울 쿠키를 하나 만들어서 MaxAge로 유효기간을 0으로 설정하여, 전달하면 된다.

그럼 기존의 쿠키가 덮어씌워지고 바로 만료가 되기 때문에 로그아웃이 이루어진다.

</br>

---

## **_Cookie 방식의 문제점_**

위와 같이 Cookie만을 활용하여 로그인과 같은 클라이언트의 상태를 유지하는 방식에는 많은 문제점이 존재한다.

1. 쿠키 저장소에 저장되어 있는 쿠키(데이터)는 임의로 변경할 수 있다.

2. 쿠키에 보관된 정보는 훔쳐갈 수 있다. (보안에 매우 취약)  
   -> 만약 쿠키에 개인정보와 같은 데이터가 있다면?  
   쿠키의 정보는 로컬PC에서 털릴수도, 네트워크 전송 구간에서 털릴수도 있다.

3. 해커가 쿠키를 훔쳐가면 평생 사용이 가능

</br>

위의 문제점을 해결하기 위한 대안으로는

1. 쿠키에 중요한 정보를 저장시키지 않고, 임의의 토큰(랜덤 값)만을 노출시키고, 이를 통해서 서버에서 사용자를 조회한다. 그리고 서버에서 해당 토큰 값들을 관리한다.  
   -> 이를 통해 해커가 해당 토큰값을 훔쳐가도 중요한 데이터가 아닌 의미 없는 값을 가져가게 된다.

2. 위 Cookie에서는 memberId값을 저장시키기 때문에 1이 아닌 2, 3, 4 이런식으로 서버에 전달을 할 경우에는 다른 회원으로 로그인이 이루어진다. 하지만 임의의 토큰값(UUID)으로 만들면 해커가 다른 회원의 토큰 값을 유츄하기가 매우 어렵다.

3. 해커가 토큰을 훔쳐갈 경우에도 해당 토큰의 만료시간(ex : 30분)을 지정하거나, 의심되는 IP로 접근할 경우의 해킹이 의심되는 경우에 토큰을 제거하는 등의 방식으로 보안을 지킬 수가 있다.

위 대안을 활용하기 위한 방안이 바로 Session을 활용하는 것이다.

</br>

---

## **_Session_**

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/198870545-db3e115f-9616-4b47-8955-ff792cf2f37a.png" width = 70%>
</p>

Cookie 방식과 같이 클라이언트에서 id와 password를 전달하면, 회원 저장소에서 존재하는 회원인지 판단하는 것은 동일하다.

이후에 존재하는 회원인 경우에서 서버가 관리하는 세션 저장소에 해당 회원에만 매핑이 되는 sessionId를 생성하여 Key, value형태로 저장한다.

그리고 생성한 sessionId를 쿠키로 만들어서 클라이언트에게 전달한다.

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/198870739-6997691f-b90a-453a-8ffa-de1693a94466.png" width = 70%>
</p>

로그인 이후에는 클라이언트가 요청마다 쿠키를 전달하고, 서버는 세션 저장소에서 클라이언트가 전달한 쿠키로 조회하여, 매핑되어 있는 회원의 정보를 꺼낸다.

그리고 로그인 이후에 보여줘야 할 페이지(로그인 된 회원 정보를 필요로 하는 페이지)에는 해당 정보를 활용해서 보여주면 된다.

위를 정리하면 결국 세션이라는 것은 특별한 기술이 아닌 그저 하나의 세션 저장소를 두어서 쿠키의 문제점을 개선을 할 뿐 클라이언트와 서버간에는 결국 쿠키를 활용한다는 것이다.

</br>

---

## **_세션 직접 만들기_**

위 방식대로 동작하는 세션을 직접 만들어보자.

```java

@Component
public class SessionManager {

    public static final String SESSION_COOKIE_NAME = "mySessionId";
    private Map<String ,Object> sessionStore = new ConcurrentHashMap<>();

    /**
     * 세션 생성
     * sessionId 생성(UUID -> 추정 불가 ID)
     * 세션 저장소에 sessionId에 보관할 값 저장
     * sessionId로 응답 쿠키를 생성해서 클라이언트에 전달
     */
    public void createSession(Object value, HttpServletResponse response){

        //세션 ID 생성하고, 값을 저장
        String sessionId = UUID.randomUUID().toString();
        sessionStore.put(sessionId, value);

        //쿠키 생성
        Cookie mySessionCookie = new Cookie(SESSION_COOKIE_NAME, sessionId);
        response.addCookie(mySessionCookie);
    }

    /**
     * 세션 조회
     */
    public Object getSession(HttpServletRequest request){
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);

        if(sessionCookie == null){
            return null;
        }

        return sessionStore.get(sessionCookie.getValue());
    }

    public Cookie findCookie(HttpServletRequest request, String cookieName){
        if(request.getCookies() == null){
            return null;
        }

        return Arrays.stream(request.getCookies())
                .filter(cookie -> cookie.getName().equals(cookieName))
                .findAny()
                .orElse(null);
    }

    /**
     * 세션 만료
     */
    public void expire(HttpServletRequest request){
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);

        if(sessionCookie != null){
            sessionStore.remove(sessionCookie.getValue());
        }
    }


}

```

세션을 만들고, 조회하고, 만료를 시키는 로직이다.

위 로직으로 로그인, 로그인 판별하여 보여줄 메인 페이지, 로그아웃을 구현해 보자.

**_로그인_**

```java
    @PostMapping("/login")
    public String loginV2(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletResponse response){

        if(bindingResult.hasErrors()){
            return "login/loginForm";
        }

        Member loginMember = loginService.login(form.getLoginId(), form.getPassword());

        if(loginMember == null){
            bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");//reject == globalError
            return "login/loginForm";
        }

        //로그인 성공 처리 TODO


        //SessionManager를 통해서 세션을 생성하고, 회원 데이터 보관
        sessionManager.createSession(loginMember,response);

        return "redirect:/";
    }
```

로그인 정보가 맞을 경우 sessionManager.createSession(loginMember,response)를 통해 세션을 만들고, "mySessionId"로 쿠키 이름을 지정하고, 해당 쿠키의 value로 UUID 형태로 만든 sesssionId를 넣는다.

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/198871243-789b656a-ed05-40bc-8fcf-98c2cff908de.png" width = 70%>
</p>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/198871247-7555c559-b38f-41ca-9470-31defb65f894.png" width = 70%>
</p>

로그인을 성공 했을 때 클라이언트에 전달되어 저장되는 결과 화면이다.

mySessionId의 name으로 SessionId(UUID로 생성)값이 저장된 것을 확인할 수 있다.

</br>

**_메인페이지_**

```java
    @GetMapping("/")
    public String homeLoginV2(HttpServletRequest request, Model model){

        //세션 관리자에 저장된 회원 정보 조회
        Member member = (Member) sessionManager.getSession(request);

        //로그인
        if(member == null){
            return "home";
        }

        model.addAttribute("member", member);
        return "loginHome";
    }
```

getSession()를 통해 쿠키에 저장된 mySessionId의 value에 매핑되는 member를 가져온다.

member가 존재하지 않는다면 로그인 이전의 메인페이지를, 존재한다면 로그인 이후의 페이지를 클라이언트에게 전달한다.

</br>

**_로그아웃_**

```java
    @PostMapping("/logout")
    public String logoutV2(HttpServletRequest request){
        sessionManager.expire(request);

        return "redirect:/";
    }
```

로그아웃을 할 경우에는 Cookie의 mySessionId의 value에 매핑퇴는 member를 remove시켜서 서버의 세션 저장소에서 제거한다.

</br>

---

## **_HttpSession 활용_**

서블릿은 HttpSession이라는 기능을 제공한다. 해당 기능을 통해 위의 세션기능을 지원할 뿐만 아니라 더 많은 기능과 스프링에서 자동으로 처리되는 부분들 또한 제공한다.

**_로그인_**

```java
//상수만을 쓰는 용도이므로, 추상 클래스나 인터페이스로 만들어서 인스턴스 생성을 막자
public interface SessionConst {
    String LOGIN_MEMBER = "loginMember"; //public static final 생략
}
```

```java
        //세션이 있으면 있는 세션을 반환하고, 없으면 신규 세션 생성
        HttpSession session = request.getSession();
        //세션에 로그인 회원 정보 보관
        session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);
```

로그인할 경우에 HttpSession을 활용하는 방법이다.

request 객체를 통해 Session을 얻어온다. 이후 setAttribute()를 통하여 "loginMember"의 name으로 loginMember의 정보를 저장한다.

그러면 여러 사용자가 로그인을 하는 경우 name 즉, 같은 key값으로 value를 저장하는것이 아닌가 하겠지만, 서버가 클라이언트에게 JsessionId의 값을 보면 UUID 형식으로 보내주는 것을 알 수 있다.

결국 HttpSession은 내부적으로는 UUID 형식으로 구분하고, 클라이언트의 요청에서 해당 값으로 getAttribute 내부에서 객체를 구분해서 전달한다.

</br>

**_메인페이지_**

```java
방법1
    @GetMapping("/")
    public String homeLoginV3(HttpServletRequest request, Model model){

        //false에 의해 이미 session이 존재한다면 해당 세션을 반환 아니면 null 반환
        HttpSession session = request.getSession(false);
        if(session == null){
            return "home";
        }

        //해당 세션에 클라이언트가 전달한 JSessionId로 Member 객체를 찾아 반환
        Member loginMember = (Member)session.getAttribute(SessionConst.LOGIN_MEMBER);

        //로그인
        if(loginMember == null){
            return "home";
        }

        model.addAttribute("member", loginMember);
        return "loginHome";
    }

방법2
    //@SessionAttribute를 통해 JsessionId를 통해 세션을 찾고, 해당 세션에 일치하는 Member 객체를 반환(없으면 null)
    @GetMapping("/")
    public String homeLoginV3Spring(@SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member loginMember, Model model){


        //로그인
        if(loginMember == null){
            return "home";
        }

        model.addAttribute("member", loginMember);
        return "loginHome";
    }

```

</br>

**_로그아웃_**

```java
//    @PostMapping("/logout")
    public String logoutV3(HttpServletRequest request){
        HttpSession session = request.getSession(false); //있으면 세션 반환, 없으면 생성X(null반환)

        if(session != null){
            session.invalidate(); //클라이언트가 전달한 세션을 제거
        }

        return "redirect:/";
    }
```

</br>

---

## **_Session 타임아웃 설정_**

클라이언트가 로그아웃 기능을 통해 로그아웃을 하였다면 정상적으로 세션을 제거하였기 때문에 클라이언트의 쿠키에는 남아있겠지만, 그에 매핑되는 세션은 서버에 존재하지 않기 때문에 정상적으로 로그아웃이 이루어져서 로그인상태를 해제한다.

하지만, 많은 클라이언트가 로그아웃 기능보다는 그저 브라우저를 종료를 한다.

서버는 브라우저를 종료했다는 것을 알 수가 없어서 세션정보를 그대로 저장하여 가지고 있을 것이고, 이러한 트래픽이 많이 쌓이면 결국 자원을 낭비하게 된다.

그러면 세션의 유지시간을 할당하면 로그아웃을 하지 않아도, 시간이 지나면 세션을 제거해서 로그아웃이 될 것이다. 하지만, 해당 시간이 지날때마다 로그아웃이 되어 시간마다 재로그인을 해야한다면 클라이언트의 입장에서는 매우 번거로울 것이다.

그런데 스프링은 자동으로 이러한 문제를 해결해준다.

default 값으로 1800초 즉, 30분을 세션 유지시간으로 설정하며, 이는 변경이 가능하다.

스프링은 클라이언트의 마지막 요청을 기준으로 세션의 생존 시간을 초기화한다.

즉 마지막 요청을 기준으로 30분 동안 로그인을 유지한다. 그렇기에 브라우저를 종료하여도 30분 뒤면 자동으로 로그아웃이 될 것이며, 지속적으로 요청을 보내면 생존 시간이 초기화 되니, 재로그인에 대한 걱정 또한 없는 것이다.
