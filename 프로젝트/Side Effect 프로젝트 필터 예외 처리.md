# **_Spring Security Filter 공통 예외 처리_**

해당 내용은 [사이드 이펙트](https://github.com/Side-Effect-Team/side-effect-backend) 프로젝트를 마무리하고 난 후 개인적으로 repository를 fork하여 Spring Security Filter의 공통예외처리에 대해 적용해 보는 내용입니다.

---

## **_상황_**

현재 배포되어 돌아가고 있는 버전의 필터 예외 처리는 아래와 같이 적용되어 있습니다.

```java
.addFilterBefore(new JwtFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class)
.addFilterBefore(new SecurityExceptionHandlerFilter(), JwtFilter.class)
```

위 설정은 JwtFilter 앞에 예외를 처리하는 SecurityExceptionHandlerFilter를 둠으로써, JwtFilter에서 발생하는 예외를 SecurityExceptionHandlerFilter가 잡아서 처리하도록 구현되어 있습니다.

### **_SecurityExceptionHandlerFilter 코드_**

```java
@Slf4j
public class SecurityExceptionHandlerFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        try{
            filterChain.doFilter(request, response);
        } catch (AuthException | InvalidValueException e) {
            setErrorResponse(response, e);
        }
    }

    private void setErrorResponse(HttpServletResponse response, BaseException e) throws IOException {
        ObjectMapper om = new ObjectMapper();
        ErrorCode errorCode = e.getErrorCode();
        ErrorResponse errorResponse = ErrorResponse.of(errorCode);
        response.setStatus(401);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setCharacterEncoding("UTF-8");
        response.getWriter().write(om.writeValueAsString(errorResponse));
    }
}
```

AuthException과 InvalidValueException의 예외가 발생했을 때 예외를 잡아서 에러에 대한 내용을 응답해줍니다.

두 예외는

```java
public class AuthException extends BaseException {

    public AuthException(ErrorCode errorCode) {
        super(errorCode);
    }
}


public class InvalidValueException extends BaseException {

    public InvalidValueException(ErrorCode errorCode) {
        super(errorCode);
    }
}
```

와 같이 공통예외로 정의해둔

```java
public class BaseException extends RuntimeException {

    private final ErrorCode errorCode;

    public BaseException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
    }

    public ErrorCode getErrorCode() {
        return errorCode;
    }
}
```

BaseException을 상속받아서 정의를 해둔 상황입니다.

그리고 실제로 JwtFilter에서 발생하게 되는 예외의 경우는 다음과 같이 정의되어 있습니다.

```java
    public boolean validateAccessToken(String accessToken){
        try {
            Jwts.parser().setSigningKey(authProperties.getSecret()).parseClaimsJws(accessToken);
            return false;
        } catch (UnsupportedJwtException e) {
            throw new AuthException(ErrorCode.ACCESS_TOKEN_UNSUPPORTED);
        } catch (MalformedJwtException e) {
            throw new AuthException(ErrorCode.ACCESS_TOKEN_MALFORMED);
        } catch (SignatureException e) {
            throw new AuthException(ErrorCode.ACCESS_TOKEN_SIGNATURE_FAILED);
        } catch (ExpiredJwtException e) {
            throw new AuthException(ErrorCode.ACCESS_TOKEN_EXPIRED);
        } catch (IllegalStateException | IllegalArgumentException e) {
            throw new AuthException(ErrorCode.ACCESS_TOKEN_ILLEGAL_STATE);
        }
    }
```

위는 JWT 토큰을 검증하는 단계에서 발생할 수 있는 예외들을 전부 AuthException으로 예외를 전환하여 던집니다.  
그리고 이 예외를 앞서 말한 SecurityExceptionHandlerFilter에서 잡아서 처리하게 되는 것입니다.

해당 코드는 문제없이 예외를 잡아서 적절하게 처리를 하지만, 따로 필터에서 발생하는 예외를 SecurityExceptionHandlerFilter의 setErrorResponse를 구현해서 처리하게 됩니다.

<br>

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/eb123bd3-c16c-425c-974b-38156854c876" width = 100%>

위는 액세스 토큰의 사용가능한 시간이 지났을 때 발생하는 예외입니다.  
 해당 예외는 JwtFilter에서 발생하며, SecurityExceptionHandlerFilter의 setErrorResponse가 동작되면서 반환되는 응답값입니다.

하지만 해당 프로젝트는 발생하는 예외들을 @RestControllerAdvice를 통해 하나의 클래스에서 처리를 하고있다는 것입니다. 위의 filter 예외처리 로직은 문제는 없지만 공통으로 예외를 핸들링해서 처리하기 위해 선언한 BaseException을 다른곳에서 사용하게 되며, 동일한 로직을 별도로 선언해서 사용중입니다. (setErrorResponse 메서드와 아래의 handleBusinessException 메서드의 로직은 결과적으로 동일한 형식으로 예외를 반환)

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BaseException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(BaseException e) {
        ErrorCode errorCode = e.getErrorCode();
        ErrorResponse response = ErrorResponse.of(errorCode);
        return new ResponseEntity<>(response, HttpStatus.valueOf(errorCode.getStatus()));
    }


    ...
}
```

위 로직이 예외들을 공통으로 핸들링하기 위해서 만들어 둔 클래스입니다.  
저가 하고자 하는 것은 @RestControllerAdvice를 통해 공통으로 예외처리하는 방식을 필터에서도 사용하는 것입니다.  
이러면 필터에서 발생하는 예외도 위의 클래스만 보면 되며, 동일한 로직을 별도로 구현하지 않아도 됩니다.

<br>

---

## **_공통 예외 처리 적용_**

#### **_공통 예외 처리 필터_**

```java
@Slf4j
@Component
public class SecurityExceptionHandlerFilter extends OncePerRequestFilter {

    private final HandlerExceptionResolver resolver;

    public SecurityExceptionHandlerFilter(@Qualifier("handlerExceptionResolver") HandlerExceptionResolver resolver) {
        this.resolver = resolver;
    }
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        try{
            filterChain.doFilter(request, response);
        } catch (AuthException | InvalidValueException e) {
            resolver.resolveException(request, response, null, e);
        }
    }

}
```

```
일반적으로는 서블릿 필터에서 스프링의 Bean을 주입받을 수 없다. 하지만, DelegatingFilterProxy의 등장으로, 서블릿 필터에서도 스프링의 Bean을 주입받아 사용할 수 있으며, 스프링 시큐리티는 DelegatingFilterProxy를 사용하며, 내부적으로 SecurityFilterChain을 호출한다.
```

해당 필터에서 HandlerExceptionResolver을 주입받아서 사용합니다.  
스프링에서 기본으로 제공해주는 ExceptionResolver가 3가지가 존재하며, 우선순위는 아래와 같습니다.

1. ExceptionHandlerExceptionResolver
2. ResponseStatusExceptionResolver
3. DefaultHandlerExceptionResolver

주입받은 HandlerExceptionResolver는 위 3가지 중에서 우선순위를 토대로 예외를 처리할 수 있는지를 판별합니다.  
@RestControllerAdvice가 존재하는 클래스의 메서드마다 걸어둔 @ExceptionHandler방식의 예외를 핸들링하는 부분에서는 우선순위가 제일 높은 ExceptionHandlerExceptionResolver가 사용됩니다.

즉, 위의 필터에서도 ExceptionHandlerExceptionResolver가 우선적으로 동작하게 됩니다.

필터가 동작될 떄 실행되는 로직에서는 해당 필터 이후에 발생하는 예외가 해당 필터까지 던져졌을 경우 resolveException메서드를 호출해서 HandlerExceptionResolver에게 인자로 전달해준 예외를 처리하도록 위임합니다.

우선순위에 의해서 ExceptionHandlerExceptionResolver가 동작하게 되고, @ExceptionHandler방식으로 필터에서 발생한 예외를 처리할 수 있게 됩니다.

```
HandlerExceptionResolver는 인터페이스이며, 여러 구현체가 빈으로 등록되어 동작한다.
그래서 @Qualifier으로 직접 사용할 빈을 명시해주지 않으면
Consider marking one of the beans as @Primary, updating the consumer to accept multiple beans, or using @Qualifier to identify the bean that should be consumed
와 같은 문구가 나타나며 실행이 되지 않는다.
동일한 타입의 빈이 여러개 등록되어 있기 때문이다.
```

#### **_ExceptionResolver 동작위치_**

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/230318749-5689b1a5-cab5-41ab-a306-6311c8c5e4ae.png" width = 70%>

ExceptionResolver는 위와 같이 DispatcherServlet을 통해서 예외를 처리합니다.  
즉, 위의 ExceptionHandlerFilter 는 필터에서 발생한 예외를 DispatcherServlet을 통해 ExceptionResolver를 사용하게 되는 것입니다.

그리고 @Component를 추가하여 빈으로 등록해 주었습니다.  
그 이유는 시큐리티 설정클래스에서 편하게 주입하여 사용하기 위함입니다.

#### **_설정 클래스_**

```java
@EnableWebSecurity
@RequiredArgsConstructor
@Slf4j
public class WebSecurityConfig{

    private final SecurityExceptionHandlerFilter securityExceptionHandlerFilter;

        @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
      return http
                //이전 설정 정보 생략
                .addFilterBefore(new JwtFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class)
                .addFilterBefore(securityExceptionHandlerFilter, JwtFilter.class)
                .build();
    }
}
```

위와 같이 @Component를 통해 빈으로 등록해 두었기 때문에 설정파일에서 주입받아서 addFilterBefore에 securityExceptionHandlerFilter 객체를 직접 넣어주기만 하면 됩니다. 그러면 securityExceptionHandlerFilter에서도 스프링이 알아서 HandlerExceptionResolver에 대한 빈을 주입시켜 줍니다.

</br>

---

## **_테스트_**

응답값은 동일할 것이므로, 디버깅을 통해 @ExceptionHandler(BaseException.class) 가 정의된 메서드가 잘 호출 되는지 확인해 보겠습니다.

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/8d12d99b-aed1-409a-939e-c8d50a9b29e4" width = 60%>

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/2073ec78-f9cc-43d2-b24e-8c1b80ff000e" width = 60%>

브레이크 포인트를 위와 같이 2곳으로 잡아놨습니다.

#### **_API 호출_**

이제 맨 위와 동일하게 만료된 액세스 토큰으로 토큰이 필요한 API를 요청해보겠습니다.  
그러면 JwtFilter에서 AuthException으로 예외가 전환될 것입니다.

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/5d62393b-918e-4b25-9843-1252166600de" width = 60%>

<br>

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/04acd62c-ffe5-4809-b63c-97a6ec985a9a" width = 60%>

API를 호출한 결과 정상적으로 브레이크 포인트에서 걸렸으며, errorCode와 response값 또한 기대한 값과 같다는 것을 확인할 수 있었습니다.

해당 방식을 통해서 Spring Security의 Filter에서 발생하는 예외 또한 ExceptionResolver를 통해서 공통으로 처리할 수 있음을 알 수 있었고, 예외를 하나의 클래스에서 공통으로 처리함에 따라 필터에서 예외처리를 했던 로직이 필요없어졌으며 이후 유지보수에도 좋을 것이라 생각합니다.

</br>

---

## **_(번외) 필터에서 예외 처리를 안해준다면?_**

필터에서 예외를 처리해주지 않으면 어떻게 되는지 공부한 내용입니다.

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/230080505-e46a8284-dfe2-4a33-9854-304fc9da42f4.png" width = 70%>

위는 필터 내에서 예외를 처리해주지 않고 그대로 반환하게 두었을 경우 발생하는 경우로, html형식의 에러페이지를 반환하게 됩니다.

html을 반환하는 이유는 스프링과 서블릿 컨테이너가 기본적으로 예외를 처리하는 방식에 있습니다.  
예외를 처리하는 방식은 다음과 같습니다.

1. 필터에서 발생한 예외이며, 이에 대한 예외처리를 해주지 않았기 때문에 서블릿 컨테이너까지 예외가 올라가게 된다.
2. 서블릿 컨테이너는 해당 예외를 처리하기 위해 다시 재요청을 한다. 재요청은 기본값으로 /error URL로 요청한다.
3. 스프링은 /error에 대한 요청을 처리하며, 이 때 스프링이 기본으로 제공하는 BasicErrorController가 동작하게 되어 html 형식의 기본 에러 페이지를 반환한다.

좀 더 자세히 동작하는 것을 보기 위해서 아래와 같이 에러를 처리하는 필터를 정의하고 등록을 해보았습니다.

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

그리고 이를 아래와 같이 예외에 대한 요청일 경우에 동작하도록 빈으로 등록했습니다.

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

즉, 서블릿 컨테이너까지 예외가 그대로 올라왔다면, 컨테이너는 /error 경로로 재요청을 보냅니다.  
그리고 따로 /error에 대한 처리를 커스텀하지 않았다면 BasicErrorController가 요청을 받아서 처리하게 되는 것입니다.

### **_BasicErrorController_**

BasicErrorController를 확인해 본 결과 아래와 같이 /error의 경로를 처리해주는 부분이 존재합니다.

```java
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {
    ...
}
```

이를 통해 필터에서 예외처리를 해주지 않을 경우의 동작방식을 알아봤습니다.
