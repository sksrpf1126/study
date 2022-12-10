# **_동적프록시 - JDK_**

해당 내용은 스프링 핵심 원리 - 고급편 (김영한님)의 강의를 보고 정리한 내용입니다.

---

100개의 클래스에 프록시를 적용할려면 100개의 프록시 클래스를 만들어야 하는 문제가 존재했다.  
그렇다면 하나의 프록시 클래스를 만들고, 해당 프록시를 원하는 클래스에 적용할 수 있다면 해결을 할 수 있을 것이다. 그리고 이를 해결하는 방법이 동적 프록시 기술을 활용하는 것이다.

동적 프록시 기술에는 크게 2가지를 설명한다.

1. JDK 동적 프록시 (인터페이스 제한)
2. CGLIB 기술 (구체 클래스 상속 활용)

여기서는 JDK 동적 프록시를 활용하는 방법에 대해 공부해 보고자 한다.

</br>

---

## **_JDK 동적 프록시 예제_**

JDK 동적 프록시 기술은 갖춰야할 조건이 있다. 바로 적용할 클래스가 인터페이스와의 구현관계가 존재해야 한다는 것이다.

```java
public interface AInterface {
    String call();
}
```

```java
@Slf4j
public class AImpl implements AInterface{
    @Override
    public String call() {
        log.info("A 호출");
        return "a";
    }
}
```

위와 같이 인터페이스와 해당 인터페이스를 구현하고 잇는 AImpl 클래스를 선언하였다.

</br>

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

@Slf4j
public class TimeInvocationHandler implements InvocationHandler {

    private final Object target;

    public TimeInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = method.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;

        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}
```

다음으로는 프록시가 될 클래스이며, JDK 동적 프록시를 사용하기 위해서는 InvocationHandler 인터페이스를 구현해야 한다. 패키지명을 보면 스프링에서 제공하는 것이 아니라는 것을 확인할 수 있다.

</br>

### **_테스트 코드_**

```java
    @Test
    void dynamicA(){
        AInterface target = new AImpl();

        TimeInvocationHandler handler = new TimeInvocationHandler(target);

        //JDK 프록시 생성 기술
        AInterface proxy = (AInterface) Proxy.newProxyInstance(AInterface.class.getClassLoader(), new Class[]{AInterface.class}, handler);
        proxy.call();
        log.info("targetClass={}", target.getClass());
        log.info("proxyClass={}", proxy.getClass());
    }
```

target 객체는 프록시를 적용할 대상이며, 핸들러 객체를 만들 때 프록시를 적용할 대상을 생성자를 통해 주입시킨다. (이전의 프록시와 디자인 패턴 내용과 동일)

이후, Proxy.newProxyInstance 메서드를 통해 프록시 객체를 동적으로 생성하는데, 동적으로 생성하기 위해 여러 인자를 넘겨준다.

1. AInterface.class.getClassLoader() 를 통해 프록시 객체에서 필요한 인터페이스 정보를 제공한다.
2. new Class[]{AInterface.class} 를 통해 프록시를 적용할 클래스가 구현하고 있는 인터페이스의 정보들을 넘기는데, 인터페이스는 다중 구현이 가능하므로 배열형태로 넘긴다.
3. 마지막으로 프록시 객체가 내부적으로 호출할 객체(메서드)를 넘겨준다. (handler 객체)

Object 타입으로 넘겨주지만 위와 같이 형변환을 통해서 받을 수 있다.

다음으로 만든 프록시 객체를 통해 call 메서드를 호출한다.

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/206670952-42ad56ec-6bd2-4b98-b558-bc1b308bc43d.png" width = 70%>
  </p>

동작 방식은 위와 같다.  
클라이언트가 call 메서드를 호출하면 실제 "서버" 객체가 아닌 만들어진 프록시 객체가 동작되고, 프록시 객체가 하는 일은 handler.invoke()를 호출하는 역할을 한다.  
해당 메서드가 호출이 되면, 실제 handler 객체의 invoke 메서드가 실행이 될 것이고, handler 객체에는 "서버" 객체인 target의 메서드를 호출한다.

```java
실행 결과
hello.proxy.jdkdynamic.code.TimeInvocationHandler - TimeProxy 실행
hello.proxy.jdkdynamic.code.AImpl - A 호출
hello.proxy.jdkdynamic.code.TimeInvocationHandler - TimeProxy 종료 resultTime=0
hello.proxy.jdkdynamic.JdkDynamicProxyTest - targetClass=class hello.proxy.jdkdynamic.code.AImpl
hello.proxy.jdkdynamic.JdkDynamicProxyTest - proxyClass=class com.sun.proxy.$Proxy9
```

실행결과는 동작방식대로 동작하는 것을 확인할 수 있으며, AInterface가 아닌 다른 인터페이스의 구현 클래스에도 언제든지 그에 맞게 동적으로 프록시 객체를 생성하여 사용할 수 있다.

</br>

---

## **_애플리케이션에 적용_**

JDK 동적 프록시 기술은 V1에만 적용할 수 있다. (V1에서는 Controller, Service, Repository를 인터페이스와 구현 클래스로 설계)

### **_Handler 클래스_**

```java
public class LogTraceFilterHandler implements InvocationHandler {

    private final Object target;
    private final LogTrace logTrace;
    private final String[] patterns;

    public LogTraceFilterHandler(Object target, LogTrace logTrace, String[] patterns) {
        this.target = target;
        this.logTrace = logTrace;
        this.patterns = patterns;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        //메서드 이름 필터
        String methodName = method.getName();

        //save, request, req*, *est 등
        if(!PatternMatchUtils.simpleMatch(patterns, methodName)) {
            //매칭이 안되면(로그 추적기능을 사용하지 않을 메서드)
            return method.invoke(target, args);
        }

        TraceStatus status = null;

        try {
            String message = method.getDeclaringClass().getSimpleName() + "." + method.getName() + "()";
            status = logTrace.begin(message);

            //target 호출
            Object result = method.invoke(target, args);
            logTrace.end(status);
            return result;
        }catch (Exception e){
            logTrace.exception(status, e);
            throw e;
        }
    }
}
```

핸들러 클래스는 위와 같으며, 해당 프록시 기술을 사용하는 경우 로그 추적기를 사용하지 않는 메서드마저 전부 적용시키는 단점이 있다. 이를 해결하기 위해서 String 배열인 patterns를 필드로 받고, invoke 메서드에서 패턴에 따라 적용할 대상을 구분한다.

이 외 나머지 로직은 데코레이터 패턴으로써, 부가 기능인 로그 추적기 기능을 넣었으며, 사이에 "서버" 객체의 실제 메서드를 호출하는 로직이 존재한다.

</br>

### **_빈 등록_**

```java
@Configuration
public class DynamicProxyFilterConfig {

    private static final String[] PATTERNS = {"request*", "order*", "save*"};

    @Bean
    public OrderControllerV1 orderControllerV1(LogTrace logTrace){
        OrderControllerV1Impl orderController = new OrderControllerV1Impl(orderServiceV1(logTrace));

        return (OrderControllerV1) Proxy.newProxyInstance(OrderControllerV1.class.getClassLoader(),
                new Class[]{OrderControllerV1.class},
                new LogTraceFilterHandler(orderController, logTrace, PATTERNS));
    }
}
```

Service와 Repository도 빈으로 등록하지만, 코드가 거의 비슷하기 때문에 컨트롤러 프록시를 등록하는 부분만 참고한다.

PATTERNS String 배열은 로그 추적 기능을 적용하지 않을 메서드 명을 작성한 것이고, 이후 빈으로 등록할 때 생성자의 인자로 넘겨준다.

빈으로 등록하는 로직을 보면, 우선 "서버" 객체를 담당할 target을 만들어서 넘겨주기 위해서 orderController 객체를 생성하며, 해당 객체는 내부적으로 orderService에 의존하므로 해당 빈을 주입시킨다.

다음으로 Proxy.newProxyInstance 메서드를 통해 프록시를 동적으로 생성하여 형변환을 통해 객체를 만들고 이를 수동으로 빈을 등록하여 사용한다.

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/206673864-79545504-d3f1-4b4b-9abd-f3184cc9f54d.png" width = 70%>
  </p>

전체적인 동작 구조는 위와 같으며, 점선이 바로 동적으로 생성한 프록시이다.

</br>

---

## **_정리_**

이를 통해 프록시를 적용할 클래스에 대한 프록시 클래스를 전부 만들어주지 않고, 동적으로 프록시를 만듦으로써 처음에 언급한 문제를 해결하게 되었다.  
하지만, JDK 동적 프록시 기술은 **_"인터페이스"_** 와 이를 구현한 클래스를 대상으로만 사용할 수 있기에, 구체 클래스로 정의한 V2 버전에서는 이를 사용할 수 없다.

V2버전 이후에서는 구체 클래스의 상속개념을 활용하여 동적으로 프록시를 만드는 기술(라이브러리)인 CGLIB를 동작 방식을 공부해보자.
