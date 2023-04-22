# **_Spring Security_**

프로젝트에서 인증 및 인가 방식을 Spring Security + JWT 방식을 활용하기로 하였다.  
쿠키 세션 방식이 아닌 JWT를 선택한 이유는 쿠키 세션방식으로는 간단하게라도 다른 프로젝트에서 적용해 본적이 있기도 하고, JWT방식으로 인증 및 인가를 경험삼아 사용해보고 싶어서이다.

Spring Security를 사용한 이유는 인증과 인가에 대해 어느정도 구현이 되어 있는 부분들을 커스텀마이징을 통해 쉽게 구현할 수 있으며, 인증이나 인가뿐만 아니라 보안이나 권한에 대한 부분도 스프링 하위 프레임워크 답게 어느정도 구현되어 있는부분이나 간단한 설정으로 제어할 수 있기 때문이다.

해당 내용에서는 보안쪽 부분보다는 인증이나 인가부분과 프로젝트를 진행을 하면서 Spring Security에 대해 공부한 내용을 정리한다.

---

## **_Spring Security란?_**

Spring Security는 Spring 기반의 어플리케이션에서 보안(인증, 인가, 권한 등)을 담당하는 **_"스프링 하위 프레임워크"_** 이다.

- 인증  
  인증이란, 쉽게 말하면 해당 사용자가 본인이 맞는지를 확인하는 과정이다. 대부분의 인증의 수단으로 아이디와 비밀번호를 전달하는 "로그인"을 활용한다.

- 인가  
  인증된 사용자가 해당 자원을 실행할 수 있는 권한이 있는지를 확인하는 과정이다.

추가적으로 보안에 대해서는 간단한 설정으로, CSRF에 대한 제어, CORS에 대한 제어 등 여러 보안적인 요소들 또한 제어가 가능하다고 한다.

</br>

---

## **_Spring Security 핵심 동작 방식_**

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/228559773-11e61282-ceb1-4e59-84cb-133765745510.png" width = 70%>

스프링 시큐리티는 필터기반으로 동작한다. 필터는 서블릿이 호출되기 전에 실행되는 로직이라고 이해하면 쉽다.

스프링에서는 디스패쳐 서블릿이라는 단 하나의 서블릿만을 사용하므로 필터들은 이 디스패쳐 서블릿이 호출되기 전에 실행이 되며, chain 방식으로 여러 필터가 있을 경우 하나의 필터가 동작된 이후 다음 필터를 호출하는 방식으로 실행이 된다.

위 이미지에서 Filter0과 Filter2는 서블릿 필터라고 하여, 스프링 시큐리티에서 제공하는 것이 아닌 따로 등록한 필터이다. 스프링 시큐리티는 그 사이에 DelegationgFilterProxy라고 하는 필터가 서블릿 필터들과 같이 필터로써 사용이 된다.

### **_DelegatingFilterProxy?_**

서블릿 필터는 스프링에서 관리되는 빈을 주입해서 사용할 수 없다.

- 그 이유는 서블릿 필터는 스프링 컨테이너에서 관리하는 컴포넌트가 아닌, 서블릿 컨테이너(Tomcat)에서 생성 및 관리하는 필터들이며, 실행되는 위치가 서로 다르기 때문이다.

하지만 스프링 시큐리티가 사용하는 이 DelegatingFilterProxy는 필터와 빈을 연결해주는 역할을 담당한다. 그렇기에 스프링 시큐리티의 필터에서는 스프링 빈을 주입하여서 구현할 수 있게 된다.

```
실제로는 DelegatingFilterProxy 또한 내부적으로 FilterChainProxy를 호출하게 된다.
```

### **FilterChainProxy**

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/228562364-8fd3393d-7262-4664-830c-59d93f27e028.png" width = 70%>

DelegatingFilterProxy가 호출하는 FilterChainProxy로, 위 이미지는 스프링 시큐리티 초기화 시에 기본적으로 생성되는 필터들의 목록이다.

위 필터들은 SeucirtyFilterChain을 빈으로 등록하는 과정에서 제어할 수 있으며, 사용자 정의 필터 또한 필터들의 전,후로 추가에서 사용할 수 있다.

핵심은 서블릿 필터들과 함께 단 하나의 DelegatingFilterProxy가 등록되어 사용되지만, 내부에서는 무수히 많은 필터들이 등록되어서 사용이 된다는 것이다.

더 정확한 필터동작과정은 아래를 참고

</br>

---

## **_SecurityFilterChain_**

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/228559783-b281ab0e-4936-41d6-8afd-66b6ea4885af.png" width = 100%>

필터들의 동작 순서들을 더 정확하게 표현한 이미지이다.

각 필터의 간단한 동작은 https://upsw-p.tistory.com/57 참고

위와 같이 많은 필터들에서 인증 및 인가를 거치지만 JWT방식으로 인증 및 인가를 구현하기에 앞서 핵심적으로 알아야 할 필터들은 UserNamePasswordAuthenticationFilter와 ExceptionTranslationFilter와 FilterSecurityInterceptor이다.

</br>

---

## **_Spring Security 인증 과정 (UserNamePasswordAuthenticationFilter)_**

스프링 시큐리티에서 기본으로 인증 과정을 처리해주는 필터로써 UserNamePasswordAuthenticationFilter를 제공해준다. 편의상 "기본인증필터"라고 표현하겠다.  
기본인증필터는 크게 3가지 방식으로 접근할 경우에 사용되도록 구현이 되어 있다.

- 기본 인증(http basic)
- 폼 로그인(form login)
- 다이제스트 인증

다이제스트 인증은 거의 사용이 되지 않는 것 같다. 그래서 해당 내용에서는 기본 인증과 폼 로그인을 알아본다.

### **_기본 인증_**

httpBasic().disable() 속성을 통해 사용을 안할 수 있으며, 이 외의 여러 속성으로도 설정할 수 있다.

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/228570361-48de78a7-4afd-452a-a702-3d11801c9aab.png" width = 70%>

httpBasic방식으로 인증과정을 거치도록 설정을 할 경우 위와 같이 인증이 필요한 요청인 경우 사용자이름과 비밀번호를 입력하도록 한다. 아이디와 비밀번호를 전달하면 기본인증필터가 동작되며, 기본인증필터 동작은 뒤에서 설명한다.

</br>

### **_폼 로그인_**

formLogin().disable() 속성을 통해 사용을 안할 수 있으며, 이 외의 여러 속성으로도 설정할 수 있다. 해당 방식은 로그인 페이지 경로를 설정을 해주어야 하며, 기본 값은 /login으로 설정되어 있는 것으로 알고 있다.

해당 url 경로에서 아이디와 비밀번호를 입력해서 어플리케이션으로 전달하는 경우에 기본인증필터가 동작된다.

</br>

### **_기본인증필터 동작방식_**

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/228570350-aa147314-3366-425b-82bb-2063940e36b4.png" width = 70%>

기본인증, 폼 로그인, 다이제스트 인증 이 3가지 방식으로 아이디와 비밀번호로 로그인 하는경우에는 위 스프링 시큐리티가 기본으로 설정한 인증필터인 UserNamePasswordAuthenticationFilter가 즉, 기본인증필터가 동작하게 된다.

자세한 내용은 https://dev-coco.tistory.com/174 와 https://wildeveloperetrain.tistory.com/57 를 참고

위 두 글에서 자세히 설명이 되어 있으므로, 이를 이해한 내용을 간단하게 요약해 본다.  
위 이미지의 숫자 순서와는 상관이 없으며, 내가 이해한 내용을 순서로 표현한 것이다.

1. Http 요청으로 아이디와 비밀번호가 넘어온다.

2. 인증필터가 동작이 되며, 위 3가지 방식중 하나라면 기본인증필터가 동작이 된다.

3. 기본인증필터에서는 아이디와 비밀번호를 가지고 UsernamePasswordAuthenticationToken 클래스의 첫번째 생성자를 통해 해당 인증 객체를 만든다. 해당 클래스는 Authentication 인터페이스를 구현한 AbstractAuthenticationToken 추상 클래스의 자식 클래스이다. (두번째 생성자는 인증이 완료된 이후에 사용이 된다)

4. AuthenticationManager는 인터페이스이다.

```java
public interface AuthenticationManager {

	Authentication authenticate(Authentication authentication) throws AuthenticationException;

}
```

위와 같이 하나의 메서드를 구현해야 하며, 해당 인터페이스의 구현체로 ProviderManager가 존재한다. 3번에서 만든 인증객체(첫번째 생성자 만든 객체로, 인증이 되지 않음)를 ProviderManager한테 전달하며, Authentication 타입으로 받는다. (업캐스팅)  
 ProviderManager에서는 List형태로 등록된 AuthenticationProvider들을 순회하며, 인증되지않은 객체를 전달하여 인증이 된 객체를 전달받을 경우에 반복문을 종료하게 된다.

5. AuthenticationProvider는 인터페이스로, 프로젝트에 맞게끔 구현체를 만들어서 빈으로 주입해서 사용해야 한다.

```java
public interface AuthenticationProvider {

	Authentication authenticate(Authentication authentication) throws AuthenticationException;

	boolean supports(Class<?> authentication);

}
```

예를 들면 인증되지 않은 객체에 담겨있는 아이디를 통해 유저의 정보를 조회해오고, 존재한다면 이후 비밀번호를 비교한다. 이 때 보통 DB의 비밀번호는 단방향 암호화 방식으로 암호화를 해놓기 떄문에, http 요청으로 넘어온 비밀번호를 동일한 암호화 방식으로 암호화해서 비교한다.

그리고 ID와 비밀번호가 일치한다면, UsernamePasswordAuthenticationToken의 두번째 생성자로써 인증객체를 새로 만들어서 ProviderManager한테 리턴한다.

6. 5번에서 아이디를 토대로 User정보를 조회할 때에는 userDetailService인터페이스를 구현한 구현체를 프로젝트에 맞게 구현하여서 이를 Provider 내부에서 호출한다.

```java
public interface UserDetailsService {

	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;

}
```

위 인터페이스를 구현한 구현체를 사용하여, 아이디를 통해 유저정보를 가져오도록 한다.

7. 6번에서 아이디를 통해 User정보를 가져올 때 스프링 시큐리티에서 제공해주는 인터페이스인 UserDetails가 존재하는데, 이를 구현한 Entity를 통해 정보를 가져와도 되고, DTO를 따로 만들어서 이를 구현해도 되고, 사용을 하지 않아도 된다.

8. 인증객체가 만들어졌으면, 최종적으로 기본인증필터로 전달하며, 기본인증필터는 SecurityContextHolder의 SecurityContext에 Authenticaion 타입으로 인증 객체를 저장한다.

두서없이 쓴거 같아 두서있게 정리를 해보면

```
1. Http Request(Id, password)
2. 기본인증필터
3. UsernamepasswordAuthenticationToken 첫번째 생성자를 통해 아이디와 비밀번호를 전달하여 인증받지 않은 객체 생성
4.인증받지 않은 객체를 AuthenticationManager를 구현한 ProviderManager한테 전달
5.ProviderManager 내부에서 프로젝트에 맞게 구현한 AuthenticationProvider 들을 List형태로 인증이 되거나 예외를 반활할 때까지  순회
6. Provider는 내부적으로 UserDetailService를 구현한 구현체를 통해 아이디에 맞는 사용자 정보 그리고 가져온 사용자 정보와 비밀번호를 비교하는 등으로 인증절차를 걸침
7. 인증이 완료될 경우 Provider는 두번째 생성자를 통해 인증 객체를 생성 후 기본인증필터 까지 해당 객체를 전달한다.
8. 인증필터는 SecurityContext에 Authenticaion 타입으로 인증 객체를 저장
```

즉, 스프링 시큐리티의 필터 내에서 인증이 완료가 되는 것이다.  
여기서 의문점은 인증이 완료된 사용자를 어떻게 관리하냐는 것이다.  
사실, 스프링 시큐리티는 기본적으로 세션방식으로 동작하게 된다. 즉, 인증절차가 완료가 되면 스프링시큐리티에서 관리하는 세션매니저 부분에서 인증이 완료된 사용자의 세션을 저장하고, 사용자에게도 세션정보를 전달해준다.

sessionManagement()를 통해 세션에 관한 여러 속성을 정의할 수 있다.

</br>

---

## **_SecurityContextHolder?_**

인증되었을 경우 세션형태로 관리한다면, 인증 객체를 SecurityContext에 저장한다는 말은 무슨 의미일까? 세션이랑 관련이 있는 클래스인가?

SecurityContext는 인증된 객체(Authentication 타입)를 저장하는 보관소로, ThreadLocal방식으로 저장되기 때문에 해당 쓰레드 내에서는 인증된 객체에 접근해서 사용하고 싶을때마다 사용할 수 있으며, ThreadLocal 특성상 사용이 끝나면 초기화를 해주어야 하는데 이러한 부분도 스프링 시큐리티가 자동으로 관리해준다.

SecurityContextHolder는 바로 이 SecurityContext를 저장하고 감싸는 클래스로, 3가지 전략을 가져갈 수 있는데, 이러한 부분은 https://catsbi.oopy.io/f9b0d83c-4775-47da-9c81-2261851fe0d0 에서 자세히 참고할 수 있다.

하나 알아두어야 하는것은 인증이 되었을 때 최종적으로 SecurityContext에서 HttpSession에 저장이 되며, 세션의 이름은 SPRING_SECURITY_CONTEXT 라고 한다.

```
정확히 정의하면 인증 시와 인증 후로 나뉜다.

인증 시에는 위와 같이 최종적으로 SecurityContext에 인증 객체가 담기고, 이를 HttpSession에 저장하게 된다.

이후 인증이 완료된 사용자의 요청이 들어올 경우 즉, 인증 후 과정에서는 Session정보를 토대로 Session에서 SecurityContext를 꺼내어 이를 SecurityContextHolder에 저장한다. 즉, 이후에 다른 로직에서 인증된 사용자의 정보를 가져와서 사용할 수 있다는 것이다.
```

</br>

---

## **_인증, 인가 예외 처리 방식_**

스프링 시큐리티는 특정 자원의 요청에 대해 인증이나 인가(권한이 있는가)에 대해 확인 후에 접근이 불가능하다면 예외를 발생시키고 이를 처리한다.

인증,인가를 검증하고 예외를 반환하는 곳은 스프링 시큐리티의 마지막에 위치해 있는 **_"FilterSecurityInterceptor"_** 에서 처리를 하며, 예외가 반환되어 해당 예외를 처리하는 로직이 존재하는 곳은 FilterSeucirtyInterceptor바로 위에 존재하는 ExceptionTranslationFilter이다.  
왜 바로 위에 있는가는 예외가 반환되었을 경우에 어플리케이션이 어떻게 동작되는지를 이해하면 바로 이해가 된다.

```
스프링은 예외가 발생하면 위로 올라간다. 위로 올라간다라는 것을 자세히 설명하면 보통 계층을 아래와 같이 나눴다고 가정하자.
필터 - 디스패쳐서블릿 - 인터셉터 - 컨트롤러 - 서비스 - 레포지토리
여기서 레포지토리단에서 예외가 반환되어서 처리를 안한다면? 레포지토리를 호출한 서비스로 예외가 반환될 것이며, 서비스에서도 이 예외를 처리하지 않는다면 컨트롤러로 올라갈 것이다.
이렇게 예외처리를 끝까지 해주지 않는다면 필터까지 올라가고, 최종적으로 사용자게에 예외를 반환하게 된다.(예외페이지 or 예외메시지)

필터내부에서 예외가 반한된다면 해당 필터를 호출한 이전 필터에게 예외를 반환할 것이다. 그리고 여기서 예외를 반환하는 필터는 FilterSecurityInterceptor이며, 이를 호출한 필터가 바로 ExceptionTranslationFilter이다.
```

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/228765942-31556f32-d1ff-4c0a-bb6c-f90f93f4fdae.png" width = 100%>

큰 동작방식은 위 이미지와 같다. 위 방식을 요약하면

1. 마지막 필터인 FilterSecurityInterceptor로 요청이 전달된다.
2. 해당 필터는 우선적으로 "인증"에 대한 여부를 검증하며, 기본적으로는(커스텀을 안했을 경우) SecurityContextHolder의 SecurityContext에 인증객체가 존재하는지 아닌지를 판단한다. 존재하지 않는다면 AuthenticationException 예외를 반환한다.
3. 인증이 되었다면, SecurityMetadataSource에서 해당 요청에 필요한 권한이 무엇인지를 가져오고, 해당 요청에 권한이 필요없다면(null이라면) 바로 접근을 허용하게 한다. 반대로, 권한이 필요하다면 AccessDecisionManager한테 처리를하도록 위임한다.
4. AccessDecisionManager에서는 인증 객체에 담겨있는 권한리스트들을 통해 해당 요청에 필요한 권한이 있는지를 판단하고 권한이 없다면 AccessDeniedException 예외를 반환하고, 있다면 접근을 허용한다.

여기서 주의깊게 봐야하는 부분은 예외가 반환되었을 경우로, 예외가 반환된다면 바로 위 필터인 ExceptionTranslationFilter로 전달되며, 해당 필터가 두 예외를 처리하도록 내부적으로 구현이 되어 있다.

```
만약 두 예외가 반환되었을 경우 처리하는 로직을 커스텀하고 싶다면?

인증 예외(AuthenticationException)가 발생했을 경우
스프링 시큐리티의 AuthenticationEntryPoint 인터페이스를 구현하고, 커스텀한 구현체를 securityFilterChain 속성을 정의하는 부분에
 .exceptionHandling().authenticationEntryPoint(new CustomAuthenticationEntryPoint())
 를 추가해주면 된다.

 인가 예외(AccessDeniedException)가 발생했을 경우
스프링 시큐리티의 AccessDeniedHandler 인터페이스를 구현하고, 커스텀한 구현체를 securityFilterChain 속성을 정의하는 부분에
 .exceptionHandling().accessDeniedHandler(new CustomAccessDeniedHandler())
 를 추가해주면 된다.

```

</br>

---

## **_참고_**

https://upsw-p.tistory.com/57 (filter-chain 순서)  
https://godekdls.github.io/Spring%20Security/servletsecuritythebigpicture/#92-delegatingfilterproxy (spring-security 공식 레퍼런스 번역 문서)  
https://catsbi.oopy.io/f9b0d83c-4775-47da-9c81-2261851fe0d0 (스프링 주요 아키텍처 이해)  
https://dev-coco.tistory.com/174 (기본인증필터 동작방식)  
https://wildeveloperetrain.tistory.com/57 (기본인증필터 동작방식)  
https://velog.io/@dailylifecoding/spring-security-memo-Authorization-and-FilterSecurityInterceptor (인증, 인가 검증 - FilterSecurityInterceptor)  
https://mangkyu.tistory.com/57 (필터에서 JWT 토큰 발급, 인터셉터에서 토큰 검증 방식)
