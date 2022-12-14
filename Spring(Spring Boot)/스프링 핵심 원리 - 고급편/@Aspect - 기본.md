# **_@Aspect - 기본_**

해당 내용은 스프링 핵심 원리 - 고급편 (김영한님)의 강의를 보고 정리한 내용입니다.

---

이전에 자동 프록시 생성기를 통해서 따로 빈 후처리기를 만들지 않고, 어드바이저만 빈으로 등록해두면 포인트 컷을 통해서 등록되는 빈들 중 프록시로 적용되어야 할 것들을 구분해서 등록해주었다.

이번에는 @Aspect 애노테이션을 활용하여 여러 어드바이저들을 매우 쉽게 빈으로 등록하는 방법을 설명한다.

</br>

---

## **_@Aspect 코드_**

```java
@Slf4j
@Aspect
public class LogTraceAspect {

    private final LogTrace logTrace;

    public LogTraceAspect(LogTrace logTrace) {
        this.logTrace = logTrace;
    }

    @Around("execution(* hello.proxy.app..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable{

        TraceStatus status = null;

        try {
            String message = joinPoint.getSignature().toShortString();
            status = logTrace.begin(message);

            //target 호출
            Object result = joinPoint.proceed();
            logTrace.end(status);
            return result;
        }catch (Exception e){
            logTrace.exception(status, e);
            throw e;
        }
    }

}
```

위와 같이 @Aspect 애노테이션을 클래스에 적용시키고, 메서드에는 @Around 애노테이션을 적용한다.  
@Around 애노테이션은 어드바이스를 적용할 "포인트 컷"의 역할을 하게되며, 메서드의 로직이 "어드바이스"가 된다.

위에서는 하나의 메서드에만 적용하였지만, 여러 메서드가 추가될 수 있다. (어드바이저가 여러개가 됨)

### **_빈 등록_**

```java
    @Bean
    public LogTraceAspect logTraceAspect(LogTrace logTrace){
        return new LogTraceAspect(logTrace);
    }
```

그리고 해당 클래스를 빈으로 등록하면 된다.  
이전에는 어드바이저를 빈으로 등록하였지만, 위 애노테이션을 활용하면 어드바이저가 아닌 클래스를 빈으로 등록하여 여러 어드바이저를 한꺼번에 사용이 가능하다.

</br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/207270080-cc4596fe-53a0-4eb0-9a42-31b14be6792c.png" width = 70%>
  </p>

큰 구조는 위와 같다.

</br>

---

## **_동작 원리_**

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/207270092-ac885fb5-5a07-4086-ae4e-939088901cb6.png" width = 70%>
  </p>

실제로 동작하는 원리는 위와 같다.  
 우선 스프링이 실행이 될 때(런타임) 빈으로 등록하는 과정에서 자동 프록시 생성기가 @Aspect 애노테이션이 존재하는 클래스들을 빈으로 등록하게 된다.(아래에서 자세히 설명) 그 후 2번 과정에서 해당 빈들을 전부 조회하여 @Aspect 어드바이저 빌더라는 것을 통해서 @Around 및 메서드 로직을 통해 어드바이저를 생성한다.  
 이후에는 이전 내용과 동일하게 어드바어지의 "포인트 컷"을 활용하여 프록시 객체를 생성하게 된다.

 </br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/207270103-c11c7eb9-1e87-4c74-88ae-4835b73cd6e2.png" width = 70%>
  </p>

더욱 세부적인 원리는 위 이미지와 같다. @Aspect 이외에 직접 어드바이저를 등록해서 사용하는 경우를 포함해서 설명한다.  
직접 어드바이저를 빈으로 등록하는 경우에는 스프링 컨테이너에서 찾아서 이후 프록시 객체 생성 여부의 판단에 바로 쓰이지만, 애노테이션 기반의 어드바이저들은 좀 다르게 동작하게 된다.

스프링은 어드바이저를 직접 빈으로 등록하는 경우에는 스포링 컨테이너에 저장이 되어서 이후 사용하지만 애노테이션 기반은 @Aspect 어드바이저 빌더를 통해서 **_처음 어드바이저를 조회할 때에는_** 여러 애노테이션의 설정을 통해 "어드바이저"를 생성하고, 이후에는 **_캐시_** 를 하여 내부적으로 가지고 있기 때문에 두 번 조회하는 경우에는 이미 만들어 놓은 어드바이저를 재활용하여 프록시 객체의 판단 여부로 사용한다.
