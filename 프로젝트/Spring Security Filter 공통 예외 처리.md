# **_Spring Security Filter 공통 예외 처리_**

해당 내용은 프로젝트에서 Spring Security의 필터에서의 공통 예외 처리 부분의 내용을 다룹니다.

---

## **_문제 상황_**

Spring Security를 통한 JWT 토큰 인증과 인가 방식을 구현 후 테스트 과정 중에 JWT토큰이 만료가 되거나, 유효하지 않는 경우 예외를 반환하도록 구현을 해놓은 상태에서 테스트를 해보았더니 아래와 같이 예외페이지(html)가 반환되었습니다.

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/230080505-e46a8284-dfe2-4a33-9854-304fc9da42f4.png" width = 70%>

예외가 발생한 부분은 JWT토큰과 관련된 로직을 정의해두는 JwtTokenProvider 클래스였으며, 해당 클래스와 관련된 스프링 시큐리티의 설정은 아래와 같았습니다.

```java
addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class)
```

그리고, 예외가 발생한 부분은 JwtTokenProvider 클래스 내부의 아래의 메서드 로직에서 발생되었습니다.

```java
    /**
     * throw ExpiredJwtException => 토큰이 만료된 경우
     * throw JwtException => 토큰이 유효하지 않은 경우(변경된 경우)
     * throw Exception => 이외의 예외가 발생한 경우
     */
    public String getUserId(String accessToken) {
        try {
            return Jwts.parser().setSigningKey(secretKey).parseClaimsJws(accessToken).getBody().getSubject();
        } catch (ExpiredJwtException e) {
            throw new CommonException(INVALID_REQUEST, "토큰이 만료되었습니다.");
        } catch (JwtException e) {
            throw new CommonException(INVALID_REQUEST, "유효하지 않은 토큰입니다.");
        } catch (Exception e) {
            throw new CommonException(INVALID_REQUEST, "유효하지 않은 토큰입니다.");
        }

    }
```

저는 처음 예외처리를 구현했을 때의 생각은 다음과 같았습니다.

1. 필터에서 발생한 예외이며, 이에 대한 예외처리를 해주지 않았기 때문에 서블릿 컨테이너까지 예외가 올라가게 된다.
2. 서블릿 컨테이너는 해당 예외를 처리하기 위해 다시 재요청을 한다.
3. 재요청이 이루어질 때 해당 예외에 대해 @RestControllerAdvice의 @ExceptionHandler에서 잡아서 처리를 하여 응답을 해준다.

하지만 반환된 결과는 Html 형식의 예외페이지가 반환되었습니다.  
저는 여기서 잘못된 생각에 사로잡혔습니다.  
 3번부분의 @ControllerAdvice부분에서 따로 설정을 해주어야 하나? 아니면 서블릿 컨테이너가 재요청을 할 때 요청헤더에 text/html 형식으로 요청을 하기 때문인가? 그럼 요청헤더에 대해 application/json 방식으로 변경해야 하나? 등 멍청한 생각들에 사로잡혀 잘못된 방향으로 해결을 할려고 열심히 찾아봤습니다.

당연히 해결책을 찾을 수 없었고, "그럼 에러가 발생했을 때 사용되는 필터를 등록하여 request 객체를 찍어보자" 하는 생각으로 아래와 같이 등록을 했습니다.

### **_예외가 발생했을 떄 동작할 필터_**

```java
@Slf4j
public class ErrorFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        log.info("예외가 서블릿 컨테이너까지 올라갔을 경우 실행되는 필터");

        chain.doFilter(request, response);
    }
}
```

그리고 이를 아래와 같이 빈으로 등록했습니다.

```java
@Bean
public FilterRegistrationBean urlFilter() {
    FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
    filterRegistrationBean.setFilter(new ErrorFilter());
    filterRegistrationBean.setDispatcherTypes(DispatcherType.ERROR);
    filterRegistrationBean.setOrder(Ordered.HIGHEST_PRECEDENCE);

    return filterRegistrationBean;
}
```

그리고 디버깅을 통해 request 객체를 살펴보던 중에 아래와 같은 부분을 확인했습니다.

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/230302658-79c95e6a-5be0-4974-82fa-cfb0c897f6cb.png" width = 70%>

"/error" 로 요청하는 것을 보고 이전에 인프런 강의에서 본 내용이 떠올랐습니다. 기본적으로 서블릿 컨테이너까지 예외가 올라갔을 때 서블릿 컨테이너는 기본 경로인 /error로 재요청을 보내고, 이를 따로 핸들링해주지 않을 경우 BasicErrorController가 처리를 합니다.

BasicErrorController를 확인해 본 결과 아래와 같이 /error의 경로를 처리해주는 것을 확인했습니다.

```java
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {
    ...
}
```

이를 통해 예외 페이지가 반환되는 이유를 알 수 있었습니다.  
하지만 저가 원하는 것은 필터에서 발생한 예외를 JSON형식으로 클라이언트에게 내려주는 것이었습니다.

</br>

---

## **_필터 예외 핸들링_**

이를 해결하기 위해 여러 글들을 참고하였습니다. 여기서 저는 크게 2가지 방식을 사용한다는 것을 알 수 있었습니다.

1. 예외가 발생되는 필터들의 이전에 예외를 처리하는 필터를 등록하고, 해당 필터에서 예외들을 받아서 response 객체에 직접 json형식으로 응답하기  
   -> https://samtao.tistory.com/48  
   -> https://jhkimmm.tistory.com/29  
   대표적으로 위 두 사이트에서 해당 방법으로 처리를 했습니다.

2. 예외를 발생했을 때 ExceptionResolver에게 처리를 위임  
   -> 필터에서 발생한 예외 또한 @ExceptionHandler를 통해 공통으로 처리할 수 있다.  
   -> https://beaniejoy.tistory.com/93 참고

저는 2번방법을 통해 예외처리하는 부분을 @RestControllerAdvice 부분에서 공통 처리를 할 수 있도록 구현을 했습니다.

선택의 이유는 1번 방법은 json형식으로 응답을 하지만, 응답 결과를 보면 스프링에서 공통으로 처리하는 방식의 결과가 아닌 문자열 형태로 쭉 나열되는 형식이었습니다. 이러면 예외를 공통 처리하는 부분의 의미가 없어지고, 클라이언트 측에서도 이에 대한 처리를 따로 작성해주어야 할 것 같아서 2번 방법을 택했습니다.

</br>

### **_공통으로 예외를 처리하는 필터 구현_**

현재 스프링 시큐리티의 필터에서 예외를 핸들링 해주는 부분이나, 발생하는 부분은 아래와 같습니다.

- CustomAccessDeniedHandler (FilterSecurityInterceptor에서 권한이 없을 경우 발생하는 예외 핸들링)
- CustomAuthenticationEntryPoint (FilterSecurityInterceptor에서 인증 객체가 없을 경우 발생하는 예외 핸들링)
- JWT 토큰에 대한 로직을 처리하는 JwtTokenProvider

저는 여기서 위 3가지 클래스에서 발생한 예외들을 공통 예외로 예외를 전환하도록 하고, 공통 예외를 처리하는 필터를 앞단에 위치하도록 등록을 하기로 했습니다.  
그리고 해당 필터에서 ExceptionResolver를 주입받아 @ExceptionHandler 방식으로 처리를 하면 생각한대로 동작할 것입니다.

#### **_CustomAccessDeniedHandler_**

```java
@Slf4j
public class CustomAccessDeniedHandler implements AccessDeniedHandler {

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        throw new CommonException(ROLE_NOT_EXISTS, "해당 계정은 권한이 없습니다.");
    }
}
```

#### **_CustomAccessDeniedHandler_**

```java
@Slf4j
public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        throw new CommonException(UNAUTHORIZED, "로그인이 필요한 서비스입니다.");
    }
}
```

외와 같이 인증과 권한에 대한 예외가 발생했을 때 공통예외로 전환해서 던지도록 수정했습니다.

#### **_JwtTokenProvider_**

```java
/**
 * throw ExpiredJwtException => 토큰이 만료된 경우
 * throw JwtException => 토큰이 유효하지 않은 경우(변경된 경우)
 * throw Exception => 이외의 예외가 발생한 경우
 */
public String getUserId(String accessToken) {
    try {
        return Jwts.parser().setSigningKey(secretKey).parseClaimsJws(accessToken).getBody().getSubject();
    } catch (ExpiredJwtException e) {
        throw new CommonException(INVALID_REQUEST, "토큰이 만료되었습니다.");
    } catch (JwtException e) {
        throw new CommonException(INVALID_REQUEST, "유효하지 않은 토큰입니다.");
    } catch (Exception e) {
        throw new CommonException(INVALID_REQUEST, "유효하지 않은 토큰입니다.");
    }

}
```

위는 JwtTokenProvicder의 일부분의 로직이며, 이 외의 로직에서도 예외를 던질 때 공통 예외로 전환해서 던지도록 구현해 놓았습니다.

#### **_공통 예외 처리 필터_**

```java
@Component
public class ExceptionHandlerFilter extends OncePerRequestFilter {

    private final HandlerExceptionResolver resolver;

    public ExceptionHandlerFilter(@Qualifier("handlerExceptionResolver") HandlerExceptionResolver resolver) {
        this.resolver = resolver;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {

        try {
            filterChain.doFilter(request, response);
        } catch (CommonException e) {
            resolver.resolveException(request, response, null, e);
        } catch (Exception e) {
            resolver.resolveException(request, response, null, e);
        }
    }
}
```

```
일반적으로는 서블릿 필터에서 스프링의 Bean을 주입받을 수 없다. 하지만, DelegatingFilterProxy의 등장으로, 서블릿 필터에서도 스프링의 Bean을 주입받아 사용할 수 있다.
```

해당 필터에서 HandlerExceptionResolver을 주입받아서 사용합니다.  
스프링에서 기본으로 제공해주는 ExceptionResolver가 3가지가 존재하며, 우선순위는 아래와 같습니다.

1. ExceptionHandlerExceptionResolver
2. ResponseStatusExceptionResolver
3. DefaultHandlerExceptionResolver

주입받은 HandlerExceptionResolver는 위 3가지 중에서 우선순위를 토대로 예외를 처리할 수 있는지를 판별합니다.  
@ExceptionHandler방식의 예외를 핸들링하는 부분에서는 우선순위가 제일 높은 ExceptionHandlerExceptionResolver가 사용됩니다.

즉, 위의 필터에서도 ExceptionHandlerExceptionResolver가 우선적으로 동작하게 됩니다.

필터가 동작될 떄 실행되는 로직에서는 해당 필터 이후에 발생하는 예외가 해당 필터까지 던져졌을 경우 resolveException메서드를 호출해서 HandlerExceptionResolver에게 인자로 전달해준 예외를 처리하도록 위임합니다.

우선순위에 의해서 ExceptionHandlerExceptionResolver가 동작하게 되고, @ExceptionHandler방식으로 필터에서 발생한 예외를 처리할 수 있게 됩니다.

#### **_ExceptionResolver 동작위치_**

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/230318749-5689b1a5-cab5-41ab-a306-6311c8c5e4ae.png" width = 70%>

ExceptionResolver는 위와 같이 DispatcherServlet을 통해서 예외를 처리합니다.  
즉, 위의 ExceptionHandlerFilter 는 필터에서 발생한 예외를 DispatcherServlet을 통해 ExceptionResolver를 사용하게 되는 것입니다.

#### **_필터 등록_**

마지막으로, 필터의 위치를 등록해주면 됩니다.

```java
.addFilterBefore(exceptionHandlerFilter, JwtAuthenticationFilter.class)
```

위와 같이 토큰로직 필터 및, URL 방식의 인증과 권한에 대한 필터보다 앞에 위치시킴으로써, 두 필터에서 발생하는 예외를 exceptionHandlerFilter가 받아서 처리합니다.

</br>

---

## **_테스트_**

#### **_토큰 검증_**

- 토큰 만료인 경우 필터내에 존재하는 JwtTokenProvider가 동작하게 될 것이며, 해당 예외는 필터에서 발생한 예외지만 @ExceptionHandler에 의해 예외가 처리되어야 합니다.

```java
    @ExceptionHandler(CommonException.class)
    public ResponseEntity<ErrorResponse> handleCommonException(CommonException e) {
        log.error("handleCommonException throw CommonException : {}", e);
        return ErrorResponse.toResponseEntity(e.getErrorCode(), e.getMessage());
    }
```

이며, 만료된 토큰으로 요청을 테스트 했을 경우

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/230321128-e94c9308-4e32-4ad8-8f44-bd748b924c98.png" width = 90%>

예외페이지가 아닌 @ExceptionHandler 방식으로 예외처리가 이루어졌습니다.

</br>

- 토큰 값을 임의로 수정했을 경우

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/230321131-ad904bd6-f1c7-4966-a486-a2c2d0989696.png" width = 90%>

정상적으로 작동하는 것을 확인할 수 있었습니다.

</br>

#### **_URL 방식의 인증 인가 예외 테스트_**

CustomAccessDeniedHandler 와 CustomAuthenticationEntryPoint에 대한 테스트입니다.  
잠시 테스트를 위해 메시지 맨 끝에 "테스트"를 추가하였습니다. (테스트를 안붙이면 메서드 방식의 인증 - 인가 예외 메시지랑 동일하기 때문)

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/230325954-6bdcf489-9576-4b0a-a84c-498d3d255aed.png" width = 90%>

와 같이 테스트를 위한 URL방식의 접근권한 설정을 해놨습니다.

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/230325965-04d85b0c-4af8-418b-ba52-8e01b4474d95.png" width = 90%>

토큰 없이 해당 경로로 요청을 할 경우에 인증객체가 존재하지 않아 위와 같이 예외를 반환합니다.

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/230325960-890feb5e-c00d-4601-a593-fd6bf9242b99.png" width = 90%>

ROLE_USER의 권한만 가진 사용자의 토큰으로 요청하는 경우 위와 같이 권한에 대한 예외를 반환합니다.  
필터에서 발생한 에외가 의도한 대로 처리가 되는 것을 확인했습니다.
