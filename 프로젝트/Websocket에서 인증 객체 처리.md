# ***Websocket에서 인증 객체 처리***

해당 내용은 [TakeEat](https://github.com/pickpong/takeeat) 프로젝트를 진행하며, 정리한 내용입니다.

해당 글에서는 Spring Security 기반에서의 웹소켓 통신중 인증된 사용자 정보 즉, 인증 객체를 처리하여 사용하는 방법을 정리하였습니다.   

---

## 부딪힌 문제

TakeEat 프로젝트를 진행 중 고객이 가게의 메뉴를 주문했을 시에 해당 가게로 주문 접수 알림이 가고, 반대로 가게가 고객의 주문에 대하여 취소, 수락, 완료 등의 상태에 따라 고객에게 알림이 가도록 구현을 해야했습니다.  

이러한 알림을 구현하기 위해 Websocket을 활용하기로 하였습니다. 그런데 서버에서 해당 알림에 대하여 처리를 할 때 검증해야 되는 부분이 있었습니다. 바로 해당 알림을 보내는 송신자가 우리 사이트를 이용하는 인증된 사용자인지 검증을 하는 부분이었습니다.  

일반적인 HTTP요청에 대해서는 잘 처리하고 있었지만, 웹소켓 기반에서는 동일한 방식으로는 안된다는 문제를 확인하였고 이를 해결하는 과정을 정리하기로 하였습니다.  

</br>

---

## 웹소켓이란?

일반적으로 클라이언트와 서버는 HTTP 기반으로 Request - Response 방식으로 통신을 주고 받습니다. 즉, 요청이 있어야 응답이 발생하고 또한 상태를 저장하지 않는 Stateless한 방식이기도 합니다.  
그리고 클라이언트의 요청에서 서버의 응답으로만 발생하는 `단방향` 통신 입니다.  

반대로 웹소켓은 StatueFul한 방식 즉, 상태를 저장(유지)하는 방식의 프로토콜입니다.  

그리고 처음 한번 연결을 맺은 이후에는 연결을 유지하면서 클라이언트와 서버 둘 다 서로에게 통신을 주고받는 `양방향` 통신입니다.  

<img width="100%" alt="웹소켓 통신 방식" src="https://github.com/user-attachments/assets/3e7b7a68-df58-4e93-ad0a-d0455a4854e0">

위와 같이 브라우저(클라이언트)와 서버는 처음에 연결을 맺고 이후에 양방향 통신을 진행하다가 한쪽이 통신 종료를 요청하거나 어떠한 이유로 종료가 되면 연결이 끊기게 됩니다.  

### HTTP Upgrade란?

위 이미지에서 웹소켓으로 연결을 맺을 때 Http Upgrade라는 용어가 등장합니다.  

웹소켓 통신으로 전환하고자 할 때에는 HTTP 요청으로 아래와 같은 헤더 정보를 전달합니다.  

```
GET /spring-websocket-portfolio/portfolio HTTP/1.1
Host: localhost:8080
Upgrade: websocket             ---- 1
Connection: Upgrade           
Sec-WebSocket-Key: Uc9l9TMkWGbHFD2qnFHltg==
Sec-WebSocket-Protocol: v10.stomp, v11.stomp
Sec-WebSocket-Version: 13
Origin: http://localhost:8080
```

위에서 1번의 Upgrade 헤더에 들어있는 정보가 바로 전환하고자 하는 프로토콜의 정보가 들어있는 헤더입니다.  

위와 같은 HTTP 프로토콜로 정보를 전달하면 서버 또한 101이라는 상태코드로 적절하게 응답을 해주고, 이후에 정상적으로 브라우저와 서버간에 웹소켓 연결이 이루어져 양방향 통신이 가능하게 됩니다.  

### 핵심은?

웹소켓의 정의를 정리한 이유는 결국 `HTTP와 Websocket은 전혀 다른 아키텍쳐를 가지고 있다` 라는 것입니다.  

이러한 핵심이 이후 일반적인 HTTP요청과 Websocket 요청에서의 인증 객체를 처리하고 사용하는 방식에 차이점이 존재하게 됩니다.  

</br>

---

## HTTP 요청에서의 인증 객체

Spring Security 기반에 세션 방식을 사용하는 현재 프로젝트의 경우에는 폼 로그인과 OAuth2 로그인을 통한 카카오 및 구글 로그인 총 3가지 방식으로 로그인을 할 수 있도록 구현했습니다.  

어떠한 방식이든 성공적으로 로그인을 하면 SecurityContextHolder라는 공간에 사용자의 인증 정보를 저장합니다.  

해당 공간은 Spring Security 설정을 통해서 여러 전략을 택할 수 있으며, 기본 전략은 ThreadLocal 기반으로 동작합니다.  

또한 해당 공간은 Authentication 타입으로 사용자의 정보를 저장합니다.  

이러한 구조에서 저희 프로젝트의 Controller에서 로그인된 사용자의 정보를 받기 위해 아래와 같이 구현을 하여 사용중입니다.  

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@AuthenticationPrincipal(expression = "#this == 'anonymousUser' ?  null : Member")
public @interface LoginMember {
}
```

현재 요청을 보낸 클라이언트의 세션을 확인하여 로그인이 되어 있다면 SecurityContextHolder에 Authentication 타입으로 사용자의 정보를 저장하고, @AuthenticationPrincipal 어노테이션을 통해 SecurityContextHolder에 존재하는 사용자의 정보를 꺼냅니다.  

이를 하나의 어노테이션으로 정의를 해두었고, 아래와 같이 사용중입니다.  

```java
    //화면에서 결제 완료 시 처리하는 로직
    @PreAuthorize("isAuthenticated()")
    @PostMapping("/complete")
    @ResponseBody
    public MarketOrdersResponse paymentComplete(@LoginMember Member member, @RequestBody PaymentOrderRequest paymentOrderRequest) {

        return paymentService.registerPayment(member, paymentOrderRequest);
    }
```

결제가 정상적으로 완료가 될 경우에 처리하는 Controller의 메서드이며, @LoginMember를 통해 Member 엔티티를 꺼내어서 사용중입니다.  

이러면 해당 메서드를 요청한 사용자의 정보를 자유롭게 사용할 수 있습니다.  

간단하게 설명했지만, 결국은 일반적인 HTTP요청에서 인증된 사용자의 정보를 사용하는 방법은 위와 같습니다.  

</br>

---

## Websocket 요청에서의 인증 객체

그러면 HTTP와 구조가 다른 Websocket에서는 어떻게 인증된 사용자의 정보를 사용해야 할까요?  

HTTP 방식과 동일하게 사용하면 어떻게 될까요?  

<img width="100%" src="https://github.com/user-attachments/assets/53887b49-9c29-42bd-a4d3-7fae46029c29">

위 메서드는 이용자가 결제 완료 후에 가게한테 주문정보를 실시간으로 전송하기 위해 수행이 되는 메서드입니다.  

@MessageMapping, @SendTo 어노테이션은 Websocket 통신을 처리하는 메서드임을 알 수 있는 어노테이션으로 @MessageMapping에 설정된 URL을 클라이언트가 요청하면 서버는 @SendTo에 명시된 URL을 `구독` 하고 있는 클라이언트들에게 응답을 해줍니다.  

핵심은 HTTP 요청을 처리하는 메서드와 동일하게 @LoginMember 어노테이션으로 인증된 사용자 정보를 받아서 처리할려고 하였지만 디버깅에서 볼 수 있듯이 member 객체에는 모두 null값이 할당되어 있는 것을 확인할 수 있습니다.  

이를 통해 `Websocket에서는 HTTP 방식과 동일한 방식은 사용이 불가` 하다를 알 수 있었습니다.  

### 왜 안될까??  

HTTP의 요청의 경우 Session의 정보를 토대로 사용자의 로그인 여부와 정보를 알아와서 SecurityContextHolder에 저장합니다.  
근데 SecurityContextHolder는 ThreadLocal을 기반으로 동작합니다.  

ThreadLocal은 하나의 쓰레드에서 자유롭게 쓸 수 있는 공간이라고 보면 됩니다. 그리고 이 쓰레드는 서버가 응답을 하고 나서는 Thead Pool에 반납하게 됩니다. 당연히 들어있는 사용자의 정보는 제거하겠죠.  

이러한 작업을 HTTP요청마다 이루어지는 것입니다. 어차피 Stateless하기 때문에 다음 요청에 새롭게 할당하면 될 뿐입니다.  

그럼 웹소켓은 어떨까요?  

웹소켓은 처음 연결할 때 동일하게 HTTP기반으로 연결을 맺고 이후에 Websocket기반으로 `양방향 통신`을 진행합니다.  

근데 웹소켓에서는 Thread를 어떻게 사용할까요? 연결을 맺고 끊어지기 전까지 하나의 쓰레드를 점유해서 계속 사용할까요?  

```java
 log.info("websocket Thread Name => {}", Thread.currentThread().getName());
```
위와 같이 메서드를 처리하는 쓰레드의 이름을 출력하는 로직을 추가해서 실제로 2번 실행해보면

```java
[boundChannel-11] c.b.t.controller.NotificationController  : websocket Thread Name => clientInboundChannel-11
[boundChannel-16] c.b.t.controller.NotificationController  : websocket Thread Name => clientInboundChannel-16  
```
다른 쓰레드임을 확인할 수 있습니다.  
그러면 왜 연결은 유지하면서 쓰레드는 메서드가 실행이 될 때마다 가져다쓰고 반납하고를 반복할까요?  
그냥 연결이 끊어질 때 반납하면 되지 않을까요?  

이에 대해 명확한 답은 찾지 못했습니다.  
그러나 저의 개인적인 생각으로는 연결이 끊어질 때까지 쓰레드를 점유하고 있는 것은 엄청난 속도로 쓰레드를 고갈시키는 위험한 행동이라고 생각합니다.  

수많은 사용자가 웹소켓을 연결시켜두고 끊지 않는다면 쓰레드는 부족할 것이고, 그러면 다른 요청들에 대해서는 처리할 방법이 없습니다.  

그렇다고 매번 쓰레드를 만든다거나 Thread Pool의 수량을 사용자의 트래픽보다도 항상 많이 만들어둘 수는 없다고 생각합니다.  

우리 서버 컴퓨터는 슈퍼 컴퓨터가 아니니까요  

#### 그러면 똑같이 ThreadLocal 방식으로 저장해서 쓰면 되는거 아니에요?

그러기 위해서는 매 요청마다 HTTP요청처럼 세션에 대한 정보를 제공해주어야 합니다.  

하지만 웹소켓은 처음 연결을 제외한 양방향 통신중에서는 일반적으로 간단한 데이터만 주고 받습니다.  

그렇기에 해당 메서드에서 SecurityContextHolder를 통해 인증된 사용자의 정보를 가져오는 것이 불가능한 것입니다.  

양방향 통신중에는 세션에 대한 정보가 없기에 SecurityContextHolder에 담겨지는 정보도 없는것입니다.  

</br>

---

## 그럼 인증된 사용자 정보는 어떻게 받아야 할까?

이를 알기 위해 많은 글들을 찾아봤습니다.  
하지만 대부분의 글들은 CSR 방식으로 클라이언트와 서버의 개발이 나누어져 있고, 이를 JWT방식으로 인증-인가를 구현하는 방식이었으며, 웹소켓 통신을 할 때 클라이언트에서 서버로 요청하는 부분에서 JWT 토큰 값을 같이 담아서 ChannelInterceptor 인터페이스를 구현하여 JWT 토큰 값을 읽어서 처리합니다.  

쉽게 말해 웹소켓에 대해 매 요청마다 JWT토큰 담아서 이를 매번 서버에서 가공한다고 보면 됩니다.  

하지만 저희 프로젝트는 인증-인가에 대해 세션 방식으로 처리하고 있습니다.  

그래서 세션 방식의 글을 찾던 도중에 아래의 블로그 글과 스프링 공식문서에서 해답을 찾았습니다.  

[세션 방식 참고 블로그](https://velog.io/@eora21/Spring-Websocket-Websocket-Security-%EB%A7%9B%EA%B9%94%EB%82%98%EA%B2%8C-%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EA%B8%B0)  
[스프링 공식문서](https://docs.spring.io/spring-framework/reference/web/websocket/stomp/authentication.html)  
[스프링 공식문서 - 매개변수 정보](https://docs.spring.io/spring-framework/reference/web/websocket/stomp/handle-annotations.html)  

쉽게 내용을 정리하면 처음에 Websocket 연결을 맺을 때에는 HTTP 통신이 먼저 이루어집니다. 이후 연결이 이루어지면 그 뒤에 Websocket을 기반으로 양방향 통신이 이루어지는 것이죠.  

그래서 Spring은 Websocket 연결을 맺을 때 HTTP 통신이 이루어지는 이 부분에서 HTTP Session을 웹소켓의 세션에도 paste 즉, 복사를 한다고 합니다.  

이 때에 기존 세션 내의 인증 정보 또한 같이 넘어가기 때문에 웹소켓 이전에 로그인된 사용자의 정보도 가져올 수 있다고 합니다.  

### HTTP 통신과 결국 다를게 없지 않나?

그런데 여기서 의문이 생깁니다. 그러면 결국 HTTP 통신과 다를게 없지 않나? 양방향 통신중에는 Websocket Session에 HTTP Session에 담겨 있는 인증 정보가 들어 있기 때문입니다.  

그러면 SecurityContextHolder로부터 인증 객체를 가져오는 것은 왜 안되는 거지? 

그래서 찾아본 결과
```
HTTP Session과 Websocket Session의 차이에 의해 SecurityContextHolder에 인증 객체를 담는 행위를 하는 AuthenticationProvider가 Websocket에는 관여하지 않는다.
```
를 찾을 수 있었습니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/336cca75-db96-4bbd-8bda-3fda5f18ca70" width = 80%></p>

위와 같이 Websocket의 경우에는 WebSocketHandler를 구현하는 추상 클래스인 AbstractWebSocketHandler를 보면 WebsocketSession을 사용하여 핸들링하는 것을 확인할 수 있습니다.  

그렇다면 SecurityContextHolder에 담기는 SecurityContext는 어디서 가져오는지를 찾아보기로 했습니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/eb6c93c9-f4eb-417c-861c-b3543a63704e" width = 80%></p>

SecurityContextPersistenceFilter가 바로 그 역할을 담당하는 클래스이며, doFilter 메서드를 보면 HttpServletRequest 타입의 객체로부터 HttpSesson을 얻어옵니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/79cf5342-9634-4670-8d24-987fb8dcc27d" width = 80%></p>

그리고 더 아래의 로직을 보면 SecurityContext 타입의 객체를 `this.repo.saveContext(...)`라는 saveContext 메서드를 호출하는 것을 볼 수 있습니다.  

해당 메서드의 내부 로직은 다음과 같습니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/88276aa9-f240-4563-a4cd-a8c1355465b0" width = 80%></p>

saveContext의 메서드는 SecurityContext타입의 인자와 HttpServletRequest 타입의 인자 그리고 HttpServletResponse 타입의 인자를 받아서 로직을 수행합니다.  

내부 로직에서 `this.saveContewxtHttpSession(context, request)` 메서드를 호출하며 해당 메서드는 바로 아래에 private으로 선언되어 있습니다.  

그리고 해당 메서드는 결국 `request.getSession(...)` 메서드를 통해 HttpSession 타입의 객체를 얻어와서 사용합니다.  

정리하면 Spring Security에서 인증 객체를 만들기 위해서 HttpSession을 사용하는 것을 확인할 수 있습니다.  

즉, 정리하면 Websocket Session을 사용하는 Websocket 통신의 경우에는 SecurityContextHolder로 부터 인증 정보를 얻어오는 행위는 불가능하다는 것을 알 수 있습니다.  

저가 만든 @LoginMember 또한 당연히 동작이 되지 않습니다.  

### 해결방법

해결방법이라고는 사실 크게 없습니다. 왜냐하면 Websocket 연결을 맺을 때 복사한 HTTP Session 정보를 가지고 인증 객체를 만들면 되고, 이를 이미 스프링은 해놓았습니다.  

공식문서에서 @MessageMapping에 인자로 사용할 수 있는 목록이 있는데 그 중에 `java.security.Principal` 타입을 사용할 수 있다고 합니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/adcb8468-29e8-44d9-8694-30ceb36e8e10" width = 80%></p>

그리고 Principal은 인터페이스이며 이를 상속하는 것이 바로 Authentication 타입입니다. 바로 SecurityContextHolder에 저장되는 타입인 그 Authentication의 부모의 입장이 되는 것이죠.  

우리는 이 Principal을 가지고 웹소켓 통신에서 인증 정보를 활용하면 될 뿐입니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/e5d582d8-0a67-46b5-82ad-7b4ef7ec5ab8" width = 80%></p>

위와 같이 웹소켓 요청을 처리하는 메서드에 Principal 타입의 인자만 추가하여 호출하고 디버깅을 해보면 아래와 같이 principal 객체 안에 member의 엔티티 객체가 존재하게 되고, 이 member 객체에 들어있는 정보가 바로 웹소켓 요청을 보낸 사용자의 인증 정보입니다.  

그런데 여기서 끝이 아닙니다. Principal 클래스에서는 getName 메서드밖에 사용이 불가합니다. 즉, Principal 안에 담겨있는 member 객체의 정보를 활용할 방법이 없습니다.  

### member의 정보를 활용하는 방법

기존에 HTTP 요청의 경우에 Authentication 타입을 활용하거나 아니면 안에 들어있는 사용자 정보만 꺼내서 사용할 수 있도록 어노테이션을 만들기도 합니다. 저희의 프로젝트 경우 @LoginMember 라는 어노테이션이 이에 해당됩니다.  

Websocket에서는 직접 형변환을 해서 사용하면 될 뿐입니다. Principal을 상속받는 것이 Authentication 이기 때문에 가능합니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/2bae5ad1-2302-46f7-9547-71757224b962" width = 80%></p>

위와 같이 Authentication 타입으로 메서드 인자를 받도록 합니다.  

디버깅을 보시면 authentication에 여러 값이 들어 있는 것을 확인할 수 있으며 다형성에 의해 실제 객체의 형태는 UsernamePasswordAuthenticationToken 타입입니다. 다음으로 principal 부분입니다. 해당 객체의 형태는 PrincipalDetails라는 타입이며 안에 member의 객체를 가지고 있습니다.  

authentication.getPrincipal()를 호출하면 PrincipalDetails 형태의 객체가 나오게 됩니다. 여기서 PrincipalDetails는 저가 구현한 클래스입니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/bea01c95-fa8d-49fd-80d9-4bcc8b7735dc" width = 80%></p>

해당 클래스에서 Member 객체를 가지고 있습니다. getMember 메서드를 통해 해당 객체를 얻어올 수 있습니다.  

그래서 `Member loginMember = ((PrincipalDetails) authentication.getPrincipal()).getMember();` 이런식으로 형변환을 해서 getMember()를 호출함으로써, 사용자의 인증 정보를 Member 타입의 엔티티 객체로 가져와 사용할 수 있습니다.  

### 형변환을 자동으로 하자!

위로도 충분히 사용자의 정보를 가져다 사용할 수 있지만, 문제는 형변환 과정을 거쳐야 합니다. 컨트롤러의 메서드의 인자를 Authentication 타입이 아니라 처음부터 PrincipalDetails 객체로 받을 수 있으면 좋지 않을까?

스프링은 컨트롤러의 메서드의 인자에 맞게 변환을 해주어 할당시킵니다. 이를 가능케 하는 것이 수많은 Argument Resolver 가 동작되기 때문입니다.  

그러면 해당 프로젝트의 사용자 인증 정보가 필요한 웹소켓 컨트롤러 메서드에서도 PrincipalDetails로 형변환하는 부분을 자동으로 하게끔 Argument Resolver를 커스텀해서 만들어서 등록만 해주면 됩니다.  

그러면 PrincipalDetails라는 타입의 인자를 필요로 하는 컨트롤러 메서드가 호출 되기 전에 해당 Argument Resolver가 우선 동작하게 되어 적절하게 형변환을 해주게 됩니다.  

#### Custom Argument Resolver

```java
public class PrincipalDetailWebsocketArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return PrincipalDetails.class.isAssignableFrom(parameter.getParameterType());
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, Message<?> message) throws Exception {
        Principal principal = SimpMessageHeaderAccessor.getUser(message.getHeaders());

        if (principal instanceof AbstractAuthenticationToken abstractAuthenticationToken) {
            return abstractAuthenticationToken.getPrincipal();
        }

        throw new AuthException(ErrorCode.FAIL_CONVERT);
    }
}
```
SimpMessageHeaderAccessor.getUser의 메서드 부분이 바로 Principal 타입의 객체를 가져올 수 있도록 해주는 메서드입니다.  

해당 메서드를 통해 Principal 타입의 객체를 가져오고 이후 instanceof 메서드를 통해 AbstractAuthenticationToken 타입으로 형변환이 가능한지 판단을 합니다.  

정상적으로 형변환이 가능한 경우에는 abstractAuthenticationToken의 getPrincipal 메서드를 호출하여 반환합니다.  

당연히 getPrincipal 메서드는 PrincipalDetails 타입의 객체를 반환하게 됩니다.  

```
AbstractAuthenticationToken의 자식으로 UsernamePasswordAuthenticationToken 과 OAuth2AuthenticationToken 등이 존재합니다.  

FormLogin인 경우에는 UsernamePasswordAuthenticationToken을 사용하고 OAuth2 로그인 즉, 소셜 로그인인 경우에는 OAuth2AuthenticationToken을 사용하게 됩니다.  

결국 AbstractAuthenticationToken의 자식들로 포함되기 때문에 다른 로그인인 경우에도 문제없이 반환할 수 있도록 AbstractAuthenticationToken으로 검증을 합니다.
```

### Argument Resolver 동작 테스트