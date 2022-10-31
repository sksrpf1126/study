# **_Filter_**

해당 내용은 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술(김영한님)의 강의를 보고 정리한 내용입니다.

---

앞서 했던 예제에서 로그인을 해야지만 메인 페이지에 상품 관리 버튼이 생겨 접근할 수 있었다. 그렇지만 로그인을 하지 않고, URL에 클라이언트가 /items(상품관리 URL)을 입력해도 접근이 가능하게 된다.

그러면 이와 같이 특정페이지들을 로그인을 해야지만 접근할 수 있도록 할려면 어떻게 해야할까? 모든 컨트롤러에 로그인 여부를 확인하는 것은 매우 좋지 않은 방법이다.

이때 여러 방법들이 존재하는데, Filter, Interceptor, AOP 등의 방법으로 해결할 수 있다. 보통 Web과 관련된 공통 관심사항들은 Filter나 Interceptor로 처리하고, Web과 관련없는 서버에서의 로그와 같은 내부적인 공통 관심사항은 AOP로 처리하는것이 적절한 것 같다.

해당 내용은 Filter를 통하여 위 문제를 해결하는 방법을 정리한다.

</br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/198971093-f5194fa4-c326-447e-b41e-203e0d121901.png" width = 70%>
  </p>

내용을 정리하기 전 스프링에서의 Filter, interceptor, AOP의 처리 방식이다.

서블릿 컨테이너에서 DispatchSevlet을 호출하기전에 Filter가 호출되는데, 이 때 서블릿 컨테이너가 만들어둔 request 객체와 response객체를 전달하여 실행한다.

Filter -> Servlet(Dispatcher Servlet) -> Interceptor -> AOP -> Controller

의 순서로 호출되며, 응답할 때에는 반대로 마지막에 Filter가 실행되어 클라이언트한테 응답한다.

</br>

---

## **_Filter 예제1_**

우선 Filter로 클라이언트의 모든 요청을 로그로 찍어보자.

```java
@Slf4j
public class LogFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("log filter init");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        log.info("log filter doFilter");

        //다운캐스팅(업캐스팅 된 대상에만 가능 즉, ServletRequest request에 처음에 들어올 때 이미 업캐스팅)
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();

        //클라이언트 구분 용도
        String uuid = UUID.randomUUID().toString();

        try{
            log.info("REQUEST [{}][{}]", uuid, requestURI);
            //다음 필터가 있으면 해당 필터 호출, 없으면 1. 서블릿 호출, 2. 인터셉터 and AOP (공부예정) 3. 컨트롤러 호출
            // 컨트롤러가 정상적으로 return 하면 역순으로 진행되다가 해당 필터로 다시 돌아오고나서 finally 호출
            //doFilter 없으면 서블릿, 컨트롤러 호출이 안되어 정상적으로 응답 불가
            chain.doFilter(request, response);
        }catch(Exception e){
            throw e;
        }finally {
            log.info("RESPONSE [{}][{}]", uuid, requestURI);
        }
    }

    @Override
    public void destroy() {
        log.info("log filter destroy");
    }
}
```

스프링에서 Filter를 사용하기 위해서는 Filter 인터페이스를 구현해야하며, 3개의 메서드를 오버라이딩을 한다.

init 메서드는 필터를 생성(초기화)할 때 실행되고, doFilter 메서드는 필터의 실행 로직을, destory는 filter를 제거할 때 실행된다. 근데 init()와 destroy()는 Filter 인터페이스에 default 메서드로 작성이 되어 있기 때문에 오버라이딩을 생략할 수 있다.

doFilter에서 request 객체와 response 객체를 HttpServletRequest, HttpServletResponse 형태가 아닌 ServletRequest, ServletResponse로 받는데, 그 이유는 Http 요청만이 아닌 다른 상황에서도 사용하기 위해서 다형성을 활용한 것이다. 그런데 해당 Filter는 Http 요청을 처리를 해야하므로, 다운캐스팅을 하였다.

이후 chain.doFilter()를 통해 다음 실행될 대상을 호출해야 한다. 주석의 내용과 같이 다음 필터가 있다면 필터를, 없다면 서블릿을 호출하고 해당 메서드가 없다면 다음에 실행할 대상이 없기 때문에 doFilter()의 로직만 처리하고 응답을 해버린다.

다음으로, 해당 필터를 적용하기 위해서는 등록을 해주어야 한다.

</br>

### **_Filter 등록_**

```java
@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean logFilter(){
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new LogFilter()); //만든 필터 등록
        filterRegistrationBean.setOrder(1); // 필터간 순서
        filterRegistrationBean.addUrlPatterns("/*"); //필터 처리할 URl 패턴

        return filterRegistrationBean;
    }
}
```

흔히 우리가 아는 Spring에서의 수동 Bean 등록과 동일하게 등록하면 된다. 단, FilterRegistrationBean의 구현체로 등록을 한다.(싱글톤 객체로 등록)

여기서 의문점이 생긴다. Filter는 Servlet 보다 앞단에서 동작이 되기 때문에 제일 위의 상단의 이미지에서도 스프링 영역에 존재하지 않는다. 그런데 Bean으로 등록하는 행위는 DI 컨테이너 즉, 스프링 컨테이너에 등록하는 행위인데 스프링 컨테이너의 영역 밖에서 실행이 가능한 것인가?

해당 의문에 대한 답변으로는

스프링부트는 임베디드 톰캣(내장 톰켓)의 설정을 빈으로 관리할 수 있게 지원한다.  
FilterRegistrationBean 또한 톰캣의 설정을 하는 객체 중 하나이며, 스프링부트에서 필터를 톰캣의 **_서블릿 컨텍스트_** 에 추가할 수 있도록 지원하는 빈이다.  
따라서, FilterRegistrationBean을 사용해서 필터를 추가하면 스프링의 애플리케이션 컨텍스트에 필터가 추가되는 것이 아니라 톰캣이 구동될 때 서블릿 컨텍스트에 필터를 추가하게 된다.

즉 서블릿 컨테이너의 영역에서 서블릿과 같이 필터가 존재하고 사용할 수 있는 것이다.

</br>

### **_필터 적용결과_**

```java
hello.login.web.filter.LogFilter         : log filter doFilter
hello.login.web.filter.LogFilter         : REQUEST [b4b0741e-4a26-4790-a7a6-473fb4fe0261][/items/add]
hello.login.web.item.ItemController      : errors=org.springframework.validation.BeanPropertyBindingResult: 2 errors
Field error in object 'item' on field 'price': rejected value [1]; codes [Range.item.price,Range.price,Range.java.lang.Integer,Range]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [item.price,price]; arguments []; default message [price],1000000,1000]; default message [1000에서 1000000 사이여야 합니다]
Error in object 'item': codes [totalPriceMin.item,totalPriceMin]; arguments [10000,1]; default message [null]
hello.login.web.filter.LogFilter         : RESPONSE [b4b0741e-4a26-4790-a7a6-473fb4fe0261][/items/add]
```

필터를 적용하고 나서의 실행결과이다. Filter -> Controller -> Filter 로 동작되는지 확인하기 위해서 컨트롤러단에서 일부로 에러가 발생하도록 하였다.

실행결과를 보면 생각한 순서와 동일하게 동작된다는 것을 알 수 있다.

</br>

---

## **_로그인 체크용 필터_**

이제 원래 목적이었던 Login 여부를 체크하여 안되어 있으면 로그인 페이지로 가도록 해보자.

```java
@Slf4j
public class LoginCheckFilter implements Filter {

    //로그인이 안되어도 이용할 수 있는 리스트
    private static final String[] whitelist = {"/", "/members/add", "/login", "/logout", "/css/*"};

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();

        HttpServletResponse httpResponse = (HttpServletResponse) response;

        try{
            log.info("인증 체크 필터 시작 {}", requestURI);

            if(isLoginCheckPath(requestURI)){
                //화이트 리스트에 속하지 않은 URI인 경우
                log.info("인증 체크 로직 실행{}", requestURI);
                HttpSession session = httpRequest.getSession(false);

                if(session == null ||session.getAttribute(SessionConst.LOGIN_MEMBER) == null){
                    log.info("미인증 사용자 요청 {}", requestURI);

                    //로그인으로 redirect (redirectURL을 쿼리파라미터로 전달하는 이유는 로그인 이후에 바로 해당 페이지로 넘어갈 수 있도록)
                    httpResponse.sendRedirect("/login?redirectURL=" + requestURI);
                    return;
                }
            }

            chain.doFilter(request,response);

        } catch (Exception e){
            throw e; //예외 로깅 가능 하지만, 톰캣까지 예외를 보내주어야 함
        } finally {
            log.info("인증 체크 필터 종료 {}", requestURI);
        }

    }

    /**
     * 화이트 리스트의 경우 인증 체크X
     */
    private boolean isLoginCheckPath(String requestURI){
        return !PatternMatchUtils.simpleMatch(whitelist, requestURI);
    }
}
```

화이트 리스트에는 로그인이 안되어 있어도 접근가능한 URL을 넣어둔다. 이후에 미래에 새로운 페이지나 URL이 추가되었을 대에는 해당 list에만 추가하면 된다.

로그인 페이지로 넘어갈때 쿼리 파라미터로 redirect 시킨 URL을 같이 넘겨주어서 로그인을 하고 난 후에는 해당 페이지로 넘어가지도록 한다.

해당 필터를 등록(설정)을 해야한다.

```java
    @Bean
    public FilterRegistrationBean loginCheckFilter(){
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new LoginCheckFilter()); //만든 필터 등록
        filterRegistrationBean.setOrder(2); // 필터간 순서
        filterRegistrationBean.addUrlPatterns("/*"); //필터 처리할 URl 패턴 (화이트 리스트로 거르기 때문에 전체 경로로)

        return filterRegistrationBean;
    }
```

이러면 필터가 2개가 되는데 순서에 의해서 처음에는 클라이언트의 요청에 대한 로그를 남기고, 다음으로는 해당 로그인 체크 필터가 동작이 된 다음 컨트롤러가 호출이 된다.

LogFilter -> LoginCheckFilter -> Servlet 호출(Dispatcher Servlet 동작) -> Interceptor -> AOP -> Controller(Service, Repository등의 비즈니스 로직 처리)

응답할 때에는 LoginCheckFilter 다음에 마지막으로 LogFilter가 동작된다.

다음은 Filter 방식이 아닌 Interceptor 방식으로 구현을 해볼 것이다.

보통 Spring을 사용할 때에는 interceptor로 처리하고, JSP/Servlet 방식에서는 Filter방식으로 처리한다고 한다.(?)
