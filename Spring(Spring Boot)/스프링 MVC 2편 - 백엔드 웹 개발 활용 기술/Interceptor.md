# **_Interceptor_**

해당 내용은 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술(김영한님)의 강의를 보고 정리한 내용입니다.

---

이전에 **_Servlet Ftiler_** 를 통해서 클라이언트의 요청 로그와 로그인 여부에 따른 페이지 접근제어를 하였다.

이번에는 Spring에서 Servlet Filter와는 비슷하면서도 다른 Interceptor를 활용하여 Servlet Filter를 대체해 보자.

Interceptor의 기본적인 구조는 아래와 같다.

```java
public interface HandlerInterceptor {

	default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {

		return true;
	}

	default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable ModelAndView modelAndView) throws Exception {
	}

	default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable Exception ex) throws Exception {
	}

}
```

해당 인터페이스를 임의의 클래스로 구현을 하여서 스프링에 등록하여서 사용하면 된다.

preHandle, postHandle, afterCompletion 3개의 메서드가 존재하며, 모두 default로 정의가 되어 있기에 사용하고자 하는 메서드만 오버라이딩을 할 수 있다.

</br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/199181701-f5daf20d-ea75-46ce-811b-38d2fa698dcd.png" width = 70%>
  </p>

각각의 메서드가 동작되는 부분은 위와 같다.

preHandle 메서드는 핸들러 어댑터를 호출하기 전에 실행이 되며, 그렇기에 request, response 객체 뿐만 아니라 핸들러 어댑터에 넘겨줄 handler 객체까지 받아서 사용할 수 있다.

postHandle 메서드는 핸들러 어댑터 호출 후에 호출이 되어 실행이 된다. 즉, 컨트롤러까지 정상적으로 실행이 된 후에 실행이 된다.

afterCompletion 메서드는 뷰가 렌더링이 된 후에 호출이 된다.

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/199181710-18f60024-4537-46c9-b33f-ba1b0e34a125.png" width = 70%>
  </p>

postHandle 메서드와 afterCompletion 메서드의 차이는 위 이미지와 같이 컨트롤러 단에서 예외가 발생하게 되면 postHandle 메서드는 실행이 안되지만, afterCompletion 은 항상 호출이 되기 때문에 파라미터에도 Exception ex 로 예외를 받을 수 있으며, 예외가 없다면 null이 들어간다.

  </br>

---

## **_Interceptor 코드_**

### **_요청 로그 Interceptor_**

```java
@Slf4j
public class LogInterceptor implements HandlerInterceptor {

    public static final String LOG_ID = "logId";

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String requestURI = request.getRequestURI();
        String uuid = UUID.randomUUID().toString();

        request.setAttribute(LOG_ID, uuid);

        //@RequestMapping : HandlerMethod 사용
        //정적 리소스 : ResourceHttpRequestHandler 사용
        if(handler instanceof HandlerMethod){
            HandlerMethod hm = (HandlerMethod) handler;//호출할 컨트롤러 메서드의 모든 정보가 포함되어 있다.
        }

        log.info("REQUEST [{}] [{}] [{}]", uuid, requestURI, handler);
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle [{}]", modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        String requestURI = request.getRequestURI();
        Object uuid = (String) request.getAttribute(LOG_ID);

        log.info("RESPONSE [{}] [{}] [{}]", uuid, requestURI, handler);

        if(ex != null){
            log.error(" afterCompletion error!! ", ex);
        }
    }
}
```

Filter는 doFilter메서드 하나에서만 요청과 컨트롤러 처리 이후의 응답부분이 같이 처리가 되기 때문에 지역변수로 uuid 처리가 가능했지만 Interceptor는 구분이 되어 있다.

위에서는 이러한 uuid 값을 넘기기 위해서 request.setAttribute로 값을 넣어두고, 응답 부분에서 get~으로 꺼내서 사용한다.

</br>

### **_로그인 여부 Interceptor_**

```java
@Slf4j
public class LoginCheckInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();

        log.info("인증 체크 인터셉터 실행 {}", requestURI);

        HttpSession session = request.getSession();

        if(session == null || session.getAttribute(SessionConst.LOGIN_MEMBER) == null){
            log.info("미인증 사용자 요청");
            response.sendRedirect("/login?requestURL=" + requestURI);
            return false;
        }

        return true;
    }
}
```

Filter 부분과 로직은 크게 다를게 없다.

</br>

---

## **_Interceptor 등록_**

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //서블릿이랑 url패턴 적는 방식이 약간 다름
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns("/css/**", "/*.ico", "/error");//해당 경로는 인터셉터 호출X

        registry.addInterceptor(new LoginCheckInterceptor())
                .order(2)
                .addPathPatterns("/**")
                .excludePathPatterns("/", "/members/add", "/login", "/logout",
                        "/css/**", "/*.ico", "/error");
    }
}
```

Interceptor를 등록해서 사용하기 위해서는 WebMvcConfigurer 를 구현을 시켜서 addInterceptors를 오버라이딩을 하여야 한다.

@Bean은 사용하지 않고, 해당 메서드에서 위와 같이 addInterceptor로 처리를 하면 된다.

Interceptor의 장점 중 하나로는 Filter에서는 whiteList로 체크 여부를 판별했지만 Interceptor는 설정파일에서 excludePathPatterns로 구분해서 명확하게 사용할 수 있다.
