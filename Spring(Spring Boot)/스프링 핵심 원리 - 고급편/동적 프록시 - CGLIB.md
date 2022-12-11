# **_동적프록시 - CGLIB_**

해당 내용은 스프링 핵심 원리 - 고급편 (김영한님)의 강의를 보고 정리한 내용입니다.

---

인터페이스가 있는 상황에서는 JDK 동적 프록시 기술을 사용하였다. 하지만 구체 클래스가 있는 상황에서는 해당 방법으로는 불가능한데, 이럴 때 사용하는 것이 CGLIB라는 기술이다.

해당 기술은 외부 라이브러리지만, 스프링 프레임워크가 내부 소스 코드에 포함시켰기 때문에, 의존성 추가 없이 사용이 가능하다.

실제로 CGLIB 기술을 사용하는 경우는 거의 없으며, 더 편리한 ProxyFactory라는 것이 해당 기술을 더 편리하게 사용할 수 있게 도와준다고 한다. 그렇기에 어떻게 동작되는지 테스트 코드만을 통해 알아보자.

</br>

---

## **_테스트 예제_**

### **_구체 클래스 ("서버" 객체)_**

```java
@Slf4j
public class ConcreteService {

    public void call(){
        log.info("ConcreteService 호출");
    }
}
```

</br>

### **_프록시 객체가 실행항 클래스_**

```java
@Slf4j
public class TimeMethodInterceptor implements MethodInterceptor {

    private final Object target;

    public TimeMethodInterceptor(Object target) {
        this.target = target;
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = methodProxy.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;

        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}
```

CGLIB 기술을 활용하기 위해서는 MethodInterceptor 인터페이스를 구현해야 한다.

</br>

### **_TEST 코드_**

```java
    @Test
    void cglib(){
        ConcreteService target = new ConcreteService();

        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(ConcreteService.class);
        enhancer.setCallback(new TimeMethodInterceptor(target));
        ConcreteService proxy = (ConcreteService) enhancer.create();

        proxy.call();

        log.info("targetClass={}", target.getClass());
        log.info("proxyClass={}", proxy.getClass());
    }
```

target 객체는 "서버" 역할을 하는 객체이며, 프록시 객체가 실행할 실제 객체에게 주입할 객체이다.

Enhancer 객체를 통해 설정을 해주고 create 메서드를 호출하면 proxy 객체가 생성이 된다. (CGLIB 기술)  
CGLIB 기술은 구체 클래스를 상속하는 방법으로 프록시 객체를 생성하는데, 그렇기에 setSuperclass 에는 부모가 될 구체 클래스("서버" 객체)를 정해줘야 한다. 이후에 setCallback을 통해서 프록시 객체가 수행(호출)할 객체를 정해줘야 하는데, 여기서 TimeMethodInterceptor 객체를 생성해서 전달한다.

이후 create 메서드와 형변환을 통해서 proxy 객체를 생성하고 이후에는 call 메서드를 호출한다.

</br>

```java
실행 결과
hello.proxy.cglib.code.TimeMethodInterceptor - TimeProxy 실행
hello.proxy.common.service.ConcreteService - ConcreteService 호출
hello.proxy.cglib.code.TimeMethodInterceptor - TimeProxy 종료 resultTime=11
hello.proxy.cglib.CglibTest - targetClass=class hello.proxy.common.service.ConcreteService
hello.proxy.cglib.CglibTest - proxyClass=class hello.proxy.common.service.ConcreteService$$EnhancerByCGLIB$$25d6b0e3
```

실행결과를 위와 같으며, proxy.getclass()의 결과를 보면, CGLIB 기술이 적용된 것을 확인할 수 있다.

</br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/206685779-a73091e4-a121-4cc6-bfac-d3c50177ea22.png" width = 70%>
  </p>

실제 동작과정은 위와 같으며, 클라이언트가 서버 객체의 call 메서드를 호출하면 서버 객체가 아닌 프록시 객체가 대신 호출되며 이 때 intercept 메서드를 호출한다. 해당 메서드는 프록시 객체를 생성할 때 전달해준 객체이다.

해당 객체 내부에서는 또 "서버" 객체에 존재하는 메서드를 호출하는 형태를 가진다.

</br>

---

## **_정리_**

인터페이스일 때와 구체 클래스일 때 각기 다른 동적 프록시 기술을 사용해봤다.

CGLIB 기술은 "상속"의 개념을 그대로 사용하여 프록시 객체를 생성하기 때문에 상속의 문제를 그대로 안고간다. (final 키워드 일 경우의 문제점과 기본 생성자의 필요성 등)

이후에는 CGLIB기술을 좀 더 쉽고 효과적으로 사용할 수 있도록 해주는 ProxyFactory를 알아보자.
