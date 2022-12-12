# **_동적프록시 - ProxyFactory_**

해당 내용은 스프링 핵심 원리 - 고급편 (김영한님)의 강의를 보고 정리한 내용입니다.

---

앞에서 동적 프록시 기술 2개를 공부했다. 하지만 2개의 기술은 특정 상황에서만 사용할 수 있다는 문제가 존재한다.

JDK 동적 프록시 기술은 인터페이스 기반의 상황에서만 사용할 수 있으며, CGLIB 기술은 구체 클래스 기반의 상황에서만 사용할 수 있었다.

인터페이스가 있는 경우에는 JDK 동적 프록시를 적용하고, 구체 클래스가 있는 경우에는 CGLIB 동적 프록시를 적용할 수 있게 해주는 방법이 ProxyFactory 기술이다.

</br>

---

## **_ProxyFactory 원리_**

</br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/206842060-bb5c480d-bd97-4fa1-b209-44157e36033f.png" width = 70%>
  </p>

클라이언트의 요청에 따라 실행할 메서드가 존재할 것이다. 프록시 팩토리는 이 때 실행할 메서드의 클래스가 인터페이스 기반인지, 구체 클래스 기반인지를 판단하여 JDK 동적 프록시 기술 또는 CGLIB 기술을 선택하여 프록시를 생성한다.

```
클라이언트의 요청보다는 프록시 팩토리에 target을 전달하는데, 이 target 객체의 클래스 정보를 통해서 프록시 기술을 선택하는 것 같다.
```

</br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/206842062-b6a2a724-8aea-4f81-ac5c-4e3efe92a683.png" width = 70%>
  </p>

좀 더 자세한 동작방식은 위 이미지와 같다.  
target의 방식에 따라 JDK 프록시 또는 CGLIB 프록시를 생성해둔다. 이후 클라이언트의 요청이 오면 JDK 프록시로 동작되는 경우에는 hadler.invoke 메서드에 의해서 adviceinvocationHandler가 동작된다. (일반적인 JDK 동적 프록시의 경우에는 InvocationHandler를 구현해서 동작) adviceinvocationHandler가 하는 일은 그저 Advice를 호출할 뿐이다.

CGLIB 프록시의 경우에도 마찬가지로 intercept 메서드에 의해서 adviceMethodInterceptor가 동작된다. (일반적인 CGLIB 프록시의 경우에는 MethodInterceptor를 구현해서 동작) adviceMethodInterceptor가 하는일은 마찬가지로 Advice를 호출할 뿐이다.

이를 사용하는 사용자는 Advice만을 정의해서 설정만 해주면 된다. 이후에는 위 동작방식에 따라서 동적 프록시가 다른 기술이라 하여도 결국 Advice가 동작되기 떄문이다.

</br>

---

## **_테스트 코드_**

### **_인터페이스 기반 target 클래스_**

```java
@Slf4j
public class ServiceImpl implements ServiceInterface{

    @Override
    public void save() {
        log.info("save 호출");
    }

    @Override
    public void find() {
        log.info("find 호출");
    }
}
```

</br>

### **_구체 클래스 기반 target 클래스_**

```java
@Slf4j
public class ConcreteService {

    public void call(){
        log.info("ConcreteService 호출");
    }
}
```

</br>

### **_프록시 클래스_**

```java
@Slf4j
public class TimeAdvice implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        //target의 실행할 메서드를 찾아 실행해줌
        Object result = invocation.proceed();

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;

        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}
```

프록시 클래스 즉, 동적 프록시가 공통으로 실행시킬 Adivce 객체이다. MethodInterceptor를 구현하는데 패키지는 org.aopalliance.intercept.MethodInterceptor로써, 최종적으로 Advice를 상속하는 인터페이스이다. 프록시 팩토리를 사용할때는 해당 인터페이스를 구현해야 한다.

이전까지 직접 프록시를 생성하는 경우나 동적으로 프록시를 생성하는 경우에 프록시 클래스에 target을 생성자를 통해 주입시켜서 사용을 하였는데, 프록시 팩토리 방식에서는 프록시 팩토리 객체를 생성할 때 target을 주입시키기 때문에 프록시 클래스에는 이를 주입받지 않는다.

파라미터로 MethodInvocation 객체를 받는데, 해당 객체안에 target 클래스 정보(메서드까지 전부)를 포함하고 있는다.  
자세한 동작방식은 테스트 코드로 확인해보자.

</br>

### **_인터페이스 기반 프록시 테스트_**

```java
    @Test
    @DisplayName("인터페이스가 있으면 JDK 동적 프록시 사용")
    void interfaceProxy() {
        ServiceInterface target = new ServiceImpl();
        ProxyFactory proxyFactory = new ProxyFactory(target);
        proxyFactory.addAdvice(new TimeAdvice());
        ServiceInterface proxy =(ServiceInterface) proxyFactory.getProxy();

        log.info("targetClass={}",target.getClass());
        log.info("proxyClass={}",proxy.getClass());

        proxy.save();

        //직접 만든 프록시는 안되고, 프록시 팩토리로 만들었을 경우에만 가능
        assertThat(AopUtils.isAopProxy(proxy)).isTrue();
        //JDK 동적 프록시로 만들어졌는지 체크
        assertThat(AopUtils.isJdkDynamicProxy(proxy)).isTrue();
        //CGLIB를 통해 동적 프록시로 만들어졌는지 체크
        assertThat(AopUtils.isCglibProxy(proxy)).isFalse();
    }

실행 결과
INFO hello.proxy.proxyfactory.ProxyFactoryTest - targetClass=class hello.proxy.common.service.ServiceImpl
INFO hello.proxy.proxyfactory.ProxyFactoryTest - proxyClass=class com.sun.proxy.$Proxy10
INFO hello.proxy.common.advice.TimeAdvice - TimeProxy 실행
INFO hello.proxy.common.service.ServiceImpl - save 호출
INFO hello.proxy.common.advice.TimeAdvice - TimeProxy 종료 resultTime=1
```

인터페이스 기반 프록시를 target으로 지정하여 프록시 팩토리르 동적 프록시를 생성하는 테스트이다.  
프록시 팩토리 객체를 생성할 때 target 객체를 전달해주기 떄문에 Advice 클래스에는 필요하지가 않다. 단지 프록시 객체를 생성할 때 메서드에 MethodInvocation객체에 target정보를 전달한다.  
이후, getProxy 메서드를 통해 동적 프록시 객체를 생성한 후 메서드를 호출하고 테스트 검증 로직을 테스트한 후 종료한다.

실행결과를 보면 proxyClass 로그에서 동적으로 생성된 것을 확인할 수 있으며, com.sun.proxy~~~ 패키지에서 생성된 것을 보아 JDK 동적 프록시 기술이 사용된 것을 확인할 수 있다.

</br>

### **_구체 클래스 기반 프록시 테스트_**

```java
    @Test
    @DisplayName("구체 클래스만 있으면 CGLIB 사용")
    void concreteProxy() {
        ConcreteService target = new ConcreteService();
        ProxyFactory proxyFactory = new ProxyFactory(target);
        proxyFactory.addAdvice(new TimeAdvice());
        ConcreteService proxy =(ConcreteService) proxyFactory.getProxy();

        log.info("targetClass={}",target.getClass());
        log.info("proxyClass={}",proxy.getClass());

        proxy.call();

        //직접 만든 프록시는 안되고, 프록시 팩토리로 만들었을 경우에만 가능
        assertThat(AopUtils.isAopProxy(proxy)).isTrue();
        //JDK 동적 프록시로 만들어졌는지 체크
        assertThat(AopUtils.isJdkDynamicProxy(proxy)).isFalse();
        //CGLIB를 통해 동적 프록시로 만들어졌는지 체크
        assertThat(AopUtils.isCglibProxy(proxy)).isTrue();
    }

실행 결과
INFO hello.proxy.proxyfactory.ProxyFactoryTest - targetClass=class hello.proxy.common.service.ConcreteService
INFO hello.proxy.proxyfactory.ProxyFactoryTest - proxyClass=class hello.proxy.common.service.ConcreteService$$EnhancerBySpringCGLIB$$ec778576
INFO hello.proxy.common.advice.TimeAdvice - TimeProxy 실행
INFO hello.proxy.common.service.ConcreteService - ConcreteService 호출
INFO hello.proxy.common.advice.TimeAdvice - TimeProxy 종료 resultTime=18
```

target만 구체 클래스 객체로 전달하여 프록시 팩토리 객체를 생성하고 나머지 코드는 동일하다.  
실행 결과에서 proxyClass 로그를 통해 CGLIB 기술로 동적 프록시를 생성한 것을 확인할 수 있다.

```
인터페이스 기반인지 구체클래스 기바인지는 상관하지 않고 CGLIB 기술로 통일해서 동적 프록시를 사용하고 싶은 경우에는 getProxy 메서드를 호출하기 전 proxyFactory.setProxyTargetClass(true) 로직을 통해 프록시 팩토리 객체에 설정값을 전달하여 사용할 수 있다.

참고
스프링 부트에서 AOP를 적용할 때 프록시가 전부 CGLIB로 만들어지는데 그 이유는 setProxyTargetClass=true 로 기본값이 설정되어 있기 때문이다.
```

</br>

---

## **_포인트 컷, 어드바이스, 어드바이저_**

지금까지 프록시 팩토리를 사용했던 방법은 매우 간단한 방식으로만 사용한 것이다.  
좀 더 깊게 이해하고 사용하기 위해서는 "포인트 컷", "어드바이스", "어드바이저" 라는 용어의 이해와 함께 사용해야 하는데, 해당 용어는 스프링 AOP에서 많이 들어보는 용어이다. 김영한 선생님의 강의방식을 보면 프록시 팩토리와 스프링 AOP는 밀접한 관련이 있을 것이며, 이후 AOP 학습을 위해 해당 용어는 반드시 이해하고 넘어가야 한다.

### **_포인트 컷_**

포인트 컷(Pointcut) 이란 어디에 부가 기능을 적용할지, 안할지를 판단하는 필터링 로직이다. 이전에는 String 배열의 patterns에 적용할 메서드 명만을 문자열로 지정하여서 구분하여 필터링 하였지만 프록시 팩토리에서는 해당 로직을 활용하여 이를 구분할 수 있다.

</br>

### **_어드바이스_**

이전에 테스트 예제에서 addAdivce를 통해 advice 객체를 추가하였으며, 해당 객체의 메서드에 작성한 로직이 동적 프록시가 생성되어서 실행할 로직이었다. 그 로직이 바로 "어드바이스" 이다.

</br>

### **_어드바이저_**

어드바이저는 단순하게 포인트 컷 하나와 어드바이스 하나를 합친 것을 의미한다.

</br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/206853333-73e077f2-d5f0-41a0-9907-103eea618589.png" width = 70%>
  </p>

프록시 팩토리에서 만든 동적 프록시는 위와 같이 동작하게 된다.  
 클라이언트의 호출에 의해 프록시 객체가 동작될 때 먼저 Pointcut 필터를 통해 해당 호출에 대해서 프록시 객체가 동작을 해야할지 말지를 판단한다. 하지 않는다면 즉시 target객체가 실행될 것이고, 해야한다면 프록시 객체가 우선적으로 실행될 것이다.

</br>

---

## **_어드바이저 테스트 예제_**

```java
    @Test
    void advisorTest1(){
        ServiceInterface target = new ServiceImpl();
        ProxyFactory proxyFactory = new ProxyFactory(target);
        DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(Pointcut.TRUE, new TimeAdvice());
        proxyFactory.addAdvisor(advisor);
        ServiceInterface proxy =(ServiceInterface) proxyFactory.getProxy();

        proxy.save();
        proxy.find();
    }
```

직접 만든 포인트 컷을 적용하기 전에 우선적으로 어드바이저를 사용하는 방법을 보자.  
DefaultPointcutAdvisor 는 가장 기본적인 Advisor 구현체로써, 하나의 포인트 컷과 하나의 어드바이스를 주입시키면 된다. 어드바이스는 이전에 사용했던 것을 포인트 컷은 Pointcut.TRUE를 통해 모든 메서드를 필터 없이 실행시킨다는 의미로 전달한다.

생성한 advisor 객체를 프록시 팩토리에 addAdvisor 메서드를 통해 추가하고 이후에는 동일하게 프록시 객체를 얻고서 실행시킨다.

사실 프록시 팩토리는 프록시를 어드바이저를 기반으로 동작하게 된다. 이전에 addAdvice 메서드 또한 사실 내부적으로 new DefaultPointcutAdvisor(Pointcut.TRUE, advice); 가 최종적으로 동작하게 되어 어드바이저로 생성해서 동작하게 된다.

</br>

### **_직접 만든 포인트 컷 적용_**

```java
    static class MyPointcut implements Pointcut {

        @Override
        public ClassFilter getClassFilter() {
            return ClassFilter.TRUE;
        }

        @Override
        public MethodMatcher getMethodMatcher() {
            return new MyMethodMatcher();
        }
    }
```

Pointcut을 직접 만들어서 사용하기 위해서는 위와 같이 인터페이스를 구현해야 하며 getClassFilter 는 클래스를 필터하고, getMethodMatcher 메서드는 메서드를 필터한다. 두 메서드의 반환 값이 true이어야만 프록시가 동작하게 되고 위 예제에서 클래스 필터는 항상 true 값을 반환하고 메서드만을 검증한다.

</br>

```java
    static class MyMethodMatcher implements MethodMatcher {

        private String matchName = "save";

        @Override
        public boolean matches(Method method, Class<?> targetClass) {
            boolean result = method.getName().equals(matchName);
            log.info("포인트 컷 호출 method={} targetClass={}", method.getName(), targetClass);
            log.info("포인트 컷 결과 result={}", result);
            return result;
        }

        // false면 위 matches 메서드가, true면 아래 matches 메서드가 실행
        @Override
        public boolean isRuntime() {
            return false;
        }

        //해당 메서드가 사용되면 매개변수가 들어오고, 해당 매개변수에 의해 캐싱이 힘들어짐 (중요한 부분 X)
        @Override
        public boolean matches(Method method, Class<?> targetClass, Object... args) {
            return false;
        }
    }
```

메서드 명을 검증하는 메서드의 반환값은 MethodMatcher이다. 즉, 객체를 반환해야 하며 그래서 위와 같이 MethodMatcher를 구현한 클래스를 만들었다.

메서드 명은 간단하게 boolean result = method.getName().equals(matchName) 로직을 통해 "save" 이름의 메서드가 호출될 때에만 프록시가 동작하도록 검증한다. ("sav\*\*" 과 같은 패턴 사용 가능)

</br>

### **_직접 만든 포인트 컷 테스트_**

```java
    @Test
    @DisplayName("직접 만든 포인트컷 적용")
    void advisorTest2(){
        ServiceInterface target = new ServiceImpl();
        ProxyFactory proxyFactory = new ProxyFactory(target);
        DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(new MyPointcut(), new TimeAdvice());
        proxyFactory.addAdvisor(advisor);
        ServiceInterface proxy =(ServiceInterface) proxyFactory.getProxy();

        proxy.save();
        proxy.find();
    }

실행 결과
INFO hello.proxy.advisor.AdvisorTest - 포인트 컷 호출 method=save targetClass=class hello.proxy.common.service.ServiceImpl
INFO hello.proxy.advisor.AdvisorTest - 포인트 컷 결과 result=true
INFO hello.proxy.common.advice.TimeAdvice - TimeProxy 실행
INFO hello.proxy.common.service.ServiceImpl - save 호출
INFO hello.proxy.common.advice.TimeAdvice - TimeProxy 종료 resultTime=1
INFO hello.proxy.advisor.AdvisorTest - 포인트 컷 호출 method=find targetClass=class hello.proxy.common.service.ServiceImpl
INFO hello.proxy.advisor.AdvisorTest - 포인트 컷 결과 result=false
INFO hello.proxy.common.service.ServiceImpl - find 호출
```

new MyPointcut()을 통해 객체를 생성하여 어드바이저 객체를 만든 후 프록시 객체를 생성하여 사용한다.
실행 결과를 보면 save 메서드는 프록시가 적용되어 호출되었지만 find 메서드는 바로 target이 실행되었다.

</br>

### **_스프링이 제공하는 포인트 컷_**

```java
    @Test
    @DisplayName("스프링이 제공하는 포인트컷 적용")
    void advisorTest3(){
        ServiceInterface target = new ServiceImpl();
        ProxyFactory proxyFactory = new ProxyFactory(target);
        NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
        pointcut.setMappedNames("save");
        DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(pointcut, new TimeAdvice());
        proxyFactory.addAdvisor(advisor);
        ServiceInterface proxy =(ServiceInterface) proxyFactory.getProxy();

        proxy.save();
        proxy.find();
    }
```

직접 만든 포인트컷을 스프링이 제공하는 포인트컷으로 바꾸어 실행한 것으로 위 실행결과와 동일하다.  
스프링은 사용자가 직접 따로 포인트컷을 만들지 않을 정도로 많은 포인트컷 방식을 제공하기 때문에 큰 동작방식만을 이해하고 넘어가면 된다.

</br>

---

## **_다수의 어드바이저_**

다수의 어드바이저 즉, 다수의 "어드바이스"를 적용하기 위해서는 어떻게 해야할까?

우선 다수의 프록시를 생성해서 이를 적용해보자.

</br>

### **_어드바이스_**

```java
    @Slf4j
    static class Advice1 implements MethodInterceptor {

        @Override
        public Object invoke(MethodInvocation invocation) throws Throwable {
            log.info("advice1 호출");
            return invocation.proceed();
        }
    }

    @Slf4j
    static class Advice2 implements MethodInterceptor {

        @Override
        public Object invoke(MethodInvocation invocation) throws Throwable {
            log.info("advice2 호출");
            return invocation.proceed();
        }
    }
```

어드바이스 2개를 선언하여 하나의 target에 둘 다 적용을 시켜보자.

</br>

### **_여러 프록시를 활용_**

```java
    @Test
    void multiAdvisorTest1(){
        ServiceInterface target = new ServiceImpl();
        ProxyFactory proxyFactory = new ProxyFactory(target);
        DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(Pointcut.TRUE, new Advice1());
        proxyFactory.addAdvisor(advisor);
        ServiceInterface proxy1 =(ServiceInterface) proxyFactory.getProxy();

        //프록시2 생성
        ProxyFactory proxyFactory2 = new ProxyFactory(proxy1);
        DefaultPointcutAdvisor advisor2 = new DefaultPointcutAdvisor(Pointcut.TRUE, new Advice2());
        proxyFactory2.addAdvisor(advisor2);
        ServiceInterface proxy2 =(ServiceInterface) proxyFactory2.getProxy();

        proxy2.save();
    }

실행 결과
advice2 호출
advice1 호출
save 호출
```

두 어드바이스를 적용하기 위해 두 개의 프록시 객체를 생성하였다.  
구조는 client -> proxy2(advisor2) -> proxy1(advisor1) -> target 과 같다.  
proxy1 은 targer을 통해 프록시팩토리 객체를 생성하고 프록시 객체를 얻었으며, proxy2는 proxy1을 통해 프록시팩토리 객체를 생성하여서 프록시 객체를 얻었다.

실행 결과를 통해 proxy2가 동작되고, 이후 proxy1이 동작되고 최종적으로 target의 save가 동작되었다.

하지만 이러면 하나의 어드바이스를 적용하기 위해서 하나의 프록시 객체가 핗요해진다. 적용해야 할 어드바이스가 많아질수록 객체 또한 그만큼 증가하게 되는 큰 단점이 존재한다. 당연히 스프링은 이러한 큰 단점으로 구현하게 두지는 않는다.

</br>

### **_하나의 프록시에 여러 어드바이스_**

```java
    @Test
    void multiAdvisorTest2(){
        DefaultPointcutAdvisor advisor1 = new DefaultPointcutAdvisor(Pointcut.TRUE, new Advice1());
        DefaultPointcutAdvisor advisor2 = new DefaultPointcutAdvisor(Pointcut.TRUE, new Advice2());

        ServiceInterface target = new ServiceImpl();
        ProxyFactory proxyFactory = new ProxyFactory(target);

        proxyFactory.addAdvisor(advisor2);
        proxyFactory.addAdvisor(advisor1);
        ServiceInterface proxy1 =(ServiceInterface) proxyFactory.getProxy();

        proxy1.save();
    }

실행 결과
advice2 호출
advice1 호출
save 호출

```

client -> proxy -> advisor2 -> advisor1 -> target 구조를 나타낸다.

위와 같이 하나의 프록시 객체에 다수의 어드바이스를 추가해서 생성할 수 있으며, 먼저 추가한 어드바이저가 먼저 작동이 되는 구조이다.

```
스프링 AOP를 처음 공부하고 사용할 때에 오해하는 것이 AOP 적용 수마다 프록시 객체가 생성되어 동작할 것이라고 생각한다. 하지만 스프링은 최적화를 진행해서 위와 같이 하나의 프록시에 여러 어드바이저를 적용하는 방식으로 동작한다.
```

</br>

---

## **_정리_**

프록시 팩토리를 통해서 동적 프록시 기술을 추상화하여 제공을 한다. JDK 동적 프록시나 CGLIB 기술을 둘 다 따로 구현할 필요가 없어진 것이다.

하지만 아직 해결되지 않은 문제가 존재한다. 아래는 프록시를 생성하여 빈을 주입하는 코드이다.

```java
    @Bean
    public OrderControllerV1 orderControllerV1(LogTrace logTrace){
        OrderControllerV1Impl orderController = new OrderControllerV1Impl(orderServiceV1(logTrace));
        ProxyFactory factory = new ProxyFactory(orderController);
        factory.addAdvisor(getAdvisor(logTrace));
        OrderControllerV1 proxy =(OrderControllerV1) factory.getProxy();
        log.info("ProxyFactory proxy={}, target={}", proxy.getClass(), orderController.getClass());
        return proxy;
    }
```

V1컨트롤러에 대한 동적 프록시 설정이다. 어드바이저 하나만을 적용한 프록시 설정인데도 불구하고 코드가 복잡해 보인다. 이러한 설정을 적용해야 할 스프링 빈이 늘어날수록 위 코드도 계속 늘어날 것이다.

또한, 컴포넌트 스캔을 통해 자동으로 빈을 등록하는 클래스들에 대해서 위 방식으로는 프록시를 적용할 수 없다는 것이다.

이후에 "빈 후처리기"를 통해서 이를 해결해보자.
