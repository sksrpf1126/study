# **_@Aspect - AOP구현_**

해당 내용은 스프링 핵심 원리 - 고급편 (김영한님)의 강의를 보고 정리한 내용입니다.

---

이전 내용에서는 @Aspect와 @Around의 애노테이션을 활용해서 프록시를 적용하는 방법을 알아봤다. 이번에는 애노테이션을 좀 더 깊게 활용해보고자 한다.

</br>

---

## **_@Around와 @Pointcut 분리 및 여러 어드바이저 적용_**

```java
@Slf4j
@Aspect
public class AspectV3 {

    @Pointcut("execution(* hello.aop.order..*(..))")
    private void allOrder(){} //pointcout signature

    //클래스 이름 패턴이 *Service 인 것만 트랜잭션 프록시 적용
    @Pointcut("execution(* *..*Service.*(..))")
    private void allService(){}

    @Around("allOrder()")
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[log] {}", joinPoint.getSignature()); //join point 시그니처

        return joinPoint.proceed(); //target 객체 호출
    }

    //hello.aop.order 패키지와 하위 패키지 이면서 클래스 이름 패턴이 *Service
    @Around("allOrder() && allService()")
    public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
        try{
            log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
            Object result = joinPoint.proceed();
            log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
            return result;
        }catch (Exception e){
            log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
            throw e;
        }finally {
            log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
        }
    }
}
```

이전에는 @Around 애노테이션이 포인트 컷의 역할을 한다고 하였다. 하지만 위에서는 따로 @Pointcut 애노테이션을 통해서 정의를 해놓고 @Around에서는 해당 포인트컷 메서드를 호출하는 방식으로 사용하고 있다.  
즉, 위와 같이 @Around와 @Pointcut으로 분리를 할 수 있으며, 이를 통한 이점은 @Pointcut의 메서드명을 통해 해당 포인트컷에 대해 명확한 의도를 설명할 수 있으며, 재사용하기에도 편리하다는 것이다.

다음으로, doLog의 어드바이저는 특정 패키지 아래에 존재하는 클래스에 적용되는 대상이며, 해당 패키지 아래에는 2개의 클래스가 존재한다.

</br>

```java
@Slf4j
@Service
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    public void orderItem(String itemId) {
        log.info("[orderService] 실행");
        orderRepository.save(itemId);
    }
}
```

</br>

```java
@Slf4j
@Repository
public class OrderRepository {

    public String save(String itemId){
        log.info("[orderRepository] 실행");
        //저장 로직
        if(itemId.equals("ex")) {
            throw new IllegalStateException("예외 발생!");
        }
        return "ok";
    }

}
```

즉, doLog 어드바이저는 위 orderService, orderRepository에 적용이 된다는 것을 알 수 있으며, doTransaction 어드바이저는 추가적으로 클래스명이 ~~Service로 끝나는 대상에 적용되는 것을 알 수 있다. 그럼 아래와 같은 구조로 프록시가 동작되는 것을 확인할 수 있다.

</br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/207811159-fe74ed67-436b-4fc7-80ba-b0431f1264f7.png" width = 70%>
  </p>

실제로 위와 같이 동작하는지 테스트 코드를 통해 확인해보자.

</br>

```java
    @Test
    void success(){
        orderService.orderItem("itemA");
    }

실행 결과
hello.aop.order.aop.AspectV3             : [log] void hello.aop.order.OrderService.orderItem(String)
hello.aop.order.aop.AspectV3             : [트랜잭션 시작] void hello.aop.order.OrderService.orderItem(String)
hello.aop.order.OrderService             : [orderService] 실행
hello.aop.order.aop.AspectV3             : [log] String hello.aop.order.OrderRepository.save(String)
hello.aop.order.OrderRepository          : [orderRepository] 실행
hello.aop.order.aop.AspectV3             : [트랜잭션 커밋] void hello.aop.order.OrderService.orderItem(String)
hello.aop.order.aop.AspectV3             : [리소스 릴리즈] void hello.aop.order.OrderService.orderItem(String)
```

orderSerivce에 orderItem 메서드를 실행시키면, 프록시 객체가 동작되고, 포인트 컷을 통해서 어드바이저의 실행유무를 판별할 것이다. service가 실행이 될 때는 doLog, doTransaction 두 어드바이저가 모두 실행이 되며, 이후 target인 서비스 원본 객체의 로직을 실행할 것이다. 다음으로 target에서 repository를 호출하고 있는데, 이 때 repository에도 프록시가 적용된 상태여서 doLog 어드바이저가 호출된 후에 target이 실행될 것이다.  
그래서 위와 같은 실행결과가 나타난다.

</br>

---

## **_포인트 컷 분리_**

```java
public class Pointcuts {

    @Pointcut("execution(* hello.aop.order..*(..))")
    public void allOrder(){} //pointcout signature

    //클래스 이름 패턴이 *Service 인 것만 트랜잭션 프록시 적용
    @Pointcut("execution(* *..*Service.*(..))")
    public void allService(){}

    @Pointcut("allOrder() && allService()")
    public void orderAndService(){}
}
```

위와 같이 포인트 컷을 따로 모아두는 클래스로 정의를 할 수 있다. 그리고 아래와 같이 패키지명을 붙여서 사용하면 된다.

</br>

```java
    @Around("hello.aop.order.aop.Pointcuts.allOrder()")
    또는
    @Around("hello.aop.order.aop.Pointcuts.orderAndService()")
```

</br>

---

## **_어드바이저 순서_**

OrderService에는 2개의 어드바이저가 적용이 되는 상태이다. 하지만 순서는 어떻게 적용될지 직접 실행을 해봐야 아는 상황이다. 만약 어드바이저 간에 순서가 상관이 없다면 크게 신경쓸 문제가 아니지만, 만양 순서를 정의해야 한다면 어떻게 해야할까?

@Order 라는 애노테이션을 통해 숫자로 순서를 정의할 수 있는데, 여기서 하나 문제점이 존재한다. 바로 이 애노테이션은 메서드 레벨이 아니라 클래스 레벨에 정의해야 한다는 것이다. 즉 어드바이저 2개에 순서를 정의할려면 2개의 @Aspect 클래스에 따로 @Order로 순서를 정해야 한다는 것이다.

해당 강의에서는 편하게 static class 를 2개 선언해서 설명한다.

```java
@Slf4j
public class AspectV5Order {

    @Aspect
    @Order(2)
    public static class LogAspect {
        @Around("hello.aop.order.aop.Pointcuts.allOrder()")
        public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
            log.info("[log] {}", joinPoint.getSignature()); //join point 시그니처

            return joinPoint.proceed(); //target 객체 호출
        }
    }

    @Aspect
    @Order(1)
    public static class TxAspect {
        //hello.aop.order 패키지와 하위 패키지 이면서 클래스 이름 패턴이 *Service
        @Around("hello.aop.order.aop.Pointcuts.orderAndService()")
        public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
            try{
                log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
                Object result = joinPoint.proceed();
                log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
                return result;
            }catch (Exception e){
                log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
                throw e;
            }finally {
                log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
            }
        }
    }

}
```

위와 같이 Class 레벨에 @Order로 순서를 정의하면 해당 순서대로 어드바이저가 작동이 된다.

</br>

---

## **_어드바이저 종류_**

@Around : 메서드 호출 전과 후에 수행되며 가장 강력한 어드바이저이다. 조인 포인트 실행 여부 선택, 반환 값 변환, 예외 변환 등 여러 행위가 가능  
@Before : 조인 포인트 실행 이전(target 메서드 실행 전)에 실행  
@AfterReturning : 조인 포인트 정상 실행 완료후(target 메서드 정상 실행 후) 실행  
@AfterThrowing : 메서드가 예외를 던지는 경우 실행  
@After : 조인 포인트가 정상 또는 에외에 관계없이 실행(finally)

어드바이저를 어느 시점에 적용하여 실행시킬 것이냐에 따라 위와 같이 여러 종류로 나눠진다. @Around로 다른 모든 것들을 대신할 수 있긴 하다.

</br>

```java
    //hello.aop.order 패키지와 하위 패키지 이면서 클래스 이름 패턴이 *Service
    @Around("hello.aop.order.aop.Pointcuts.orderAndService()")
    public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
        try{
            //@Before
            log.info("[트랜잭션 시작] {}", joinPoint.getSignature());

            Object result = joinPoint.proceed();

            //@AfterReturning
            log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
            return result;
        }catch (Exception e){
            //@AfterThrowing
            log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
            throw e;
        }finally {
            //@After
            log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
        }
    }
```

위는 @Around로 동작하는 doTransaction 메서드인데 주석 처리한 부분들이 바로 해당 애노테이션을 사용했을 때 동작되는 시점을 표현한 것이다. (반대로 말하면 @Around는 모든 것을 대체할 수 있음)

</br>

```java
    @Before("hello.aop.order.aop.Pointcuts.orderAndService()")
    public void doBefore(JoinPoint joinPoint){
        log.info("[before] {}", joinPoint.getSignature());
    }

    @AfterReturning(value = "hello.aop.order.aop.Pointcuts.orderAndService()", returning = "result")
    public void doReturn(JoinPoint joinPoint, Object result){
        log.info("[AfterReturning] {} result={}", joinPoint.getSignature(), result);
    }

    @AfterReturning(value = "hello.aop.order.aop.Pointcuts.allOrder()", returning = "result")
    public void doReturn2(JoinPoint joinPoint, String result){
        log.info("[AfterReturning2] {} result={}", joinPoint.getSignature(), result);
    }

    @After(value = "hello.aop.order.aop.Pointcuts.orderAndService()")
    public void doAfter(JoinPoint joinPoint){
        log.info("[after] {}", joinPoint.getSignature());
    }
```

자세한 동작과정을 위해서 위와 같이 여러 어드바이저들을 애노테이션으로 구분해서 정의해 놓았다. 이를 한 번 테스트 코드로 동작시켜보자.

```java
    @Test
    void success(){
        orderService.orderItem("itemA");
    }

실행 결과
 hello.aop.order.aop.AspectV6Advice       : [트랜잭션 시작] void hello.aop.order.OrderService.orderItem(String)
 hello.aop.order.aop.AspectV6Advice       : [before] void hello.aop.order.OrderService.orderItem(String)
 hello.aop.order.OrderService             : [orderService] 실행
 hello.aop.order.OrderRepository          : [orderRepository] 실행
 hello.aop.order.aop.AspectV6Advice       : [AfterReturning] void hello.aop.order.OrderService.orderItem(String) result=null
 hello.aop.order.aop.AspectV6Advice       : [after] void hello.aop.order.OrderService.orderItem(String)
 hello.aop.order.aop.AspectV6Advice       : [트랜잭션 커밋] void hello.aop.order.OrderService.orderItem(String)
 hello.aop.order.aop.AspectV6Advice       : [리소스 릴리즈] void hello.aop.order.OrderService.orderItem(String)
```

해당 테스트 코드를 돌리면 @Around 애노테이션으로 정의된 doTransaction이 동작하게 된다. 이후 @Before가 동작하게 되며 다음으로는 target이 호출이 된다.  
target 호출이 "정상적"으로 종료가 되었으므로, @AfterThrowing이 아닌 @AfterReturning이 동작하게 된다. 그리고 마지막으로 @After가 동작하게 된다.  
만약 target이 예외를 반환했다면 @AfterThrowing이 동작하게 되었을 것이다.

```
@AfterReturning에서 returning으로 정의한 문자열은 메서드 파라미터의 반환값을 받는 변수명과 일치해야하며, target의 반환타입과 해당 어드바이저의 파라미터 타입이 일치 또는 포함할 수 있어야 동작하게 된다.
예를 들어 repository가 현재는 String으로 반환하기 때문에 Object로 받으면 null로 String으로 받으면 문자열로 잘 받아서 어드바이저가 동작이 되겠지만, Integer와 같은 전혀 다른 타입으로 @AfterReturing을 정의해 놓으면 해당 어드바이저는 동작하지 않게 된다.

주의할 점은 반환타입이 매칭이 안된다고 프록시 객체로 만들지 않는 것은 아니다. 프록시 객체를 만들 때 판단하는 것은 아마 value로만 판단하는 것 같다.
```

</br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/207811168-78558dd6-af55-4131-81b0-a56831651452.png" width = 70%>
  </p>

target이 정상 호출되었다고 가정하면, 1 - 2 - 4 - 5 - 6 순으로 동작하게 될 것이며, 예외를 반환했다면 1 - 2 - 3 - 5 - 6 으로 동작하게 될 것이다.

그런데 여기서 의문점이 든다. @Around로 다른 종류의 어드바이저들의 역할을 해낼 수 있는데, 왜 위와 같이 세부적으로 나누는 것일까?  
만약 @Around로 @Before와 같이 target의 메서드 호출 전에만 동작하게 하는 어드바이저를 만들고자 한다해도, @Around에 ProceedingJoinPoint의 proceed 메서드를 호출하지 않는다면 다음에 호출되어야 할 어드바이저 혹은 target의 메서드로 넘어가지가 않는다. 즉, 문제가 발생한다는 것이다.

이럴 때 차라리 @Before로 구현해 놓는다면, 나 또는 이외의 다른 사람들과 함께 작업할 때에도 "제약"을 둠으로써, 위와 같은 사태는 벌어지지 않을 것이며, 또한 의도가 명확히 보이기 때문에 더 좋은 코드가 될 것이다.

내 개인적인 생각으로 이러한 점도 어쩌면 SRP(단일 책임 원칙)을 지키기 위해 구분한 것이라고 생각한다. (세부적으로 책임을 분리)
