# **_ThreadLocal -1 (원리)_**

해당 내용은 스프링 핵심 원리 - 고급편 (김영한님)의 강의를 보고 정리한 내용입니다.

---

로그 추적기를 직접 만들어서 사용을 할려다 보니 컨트롤러 - 서비스 - 레포지토리 3개의 계층간의 단계(level) 상태 유지를 해결해야 했다.

그래서 처음에는 각 계층의 메서드의 파라미터로 단계를 유지하기 위한 인자를 전달하였지만, 이러면 모든 메서드에 인자가 하나 추가가 되는 것이며, 유지보수에도 매우 번거로울 것이다.

메서드의 파라미터 추가와 인자값을 계속 전달해야 한다는 문제를 개선을 하기 위해서 클래스에 상태값을 저장하여 동기화하는 필드를 하나 추가를 해서 사용하는 것이다.

</br>

---

## **_동기화 필드 추가_**

### **_로그 추적기 클래스_**

```java
public class FieldLogTrace implements LogTrace{

    private static final String START_PREFIX = "-->";
    private static final String COMPLETE_PREFIX = "<--";
    private static final String EX_PREFIX = "<X-";

    private TraceId traceIdHolder; //traceId 동기화

    private void syncTraceId(){
        if(traceIdHolder == null){
            traceIdHolder = new TraceId();
        } else {
            traceIdHolder = traceIdHolder.createNextId();
        }
    }

    private void releaseTraceId() {
        if(traceIdHolder.isFirstLevel()){
            traceIdHolder = null;
        } else {
            traceIdHolder = traceIdHolder.createPreviousId();
        }
    }
}
```

핵심적으로 봐야할 부분만을 남겼다.  
TraceId 인스턴스를 참조하는 참조변수 traceIdHolder를 필드로 선언하였고, 해당 필드가 바로 상태를 동기화하기 위해 사용되는 필드이다.

컨트롤러에서 로그추적기가 최초로 실행될 때 releaseTraceId()를 통해 상태값의 여부에 따라 필드에 값을 넣어준다.

이후 다른 계층의 메서드가 호출될 때 마다 지속적으로 releaseTraceId()가 호출됨에 따라 다음 단계의 상태값을 저장하게 된다.

반대로 계층의 메서드가 종료가 되기전에는 releaseTraceId()를 통해 단계의 상태값을 하나 낮춘다.

마지막으로 다시 컨트롤러 메서드가 종료가 되기전에 호출이 될 때에는 isFirstLevel()에 의해 현 상태의 레벨값이 0임을 판단하여 필드의 값을 비워준다.(null 값)

### **_Controller_**

```java
    @GetMapping("/v3/request")
    public String request(String itemId){

        TraceStatus status = null;
        try{
            status = trace.begin("OrderController.request()");
            orderService.orderItem(itemId);
            trace.end(status);
            return "ok";
        }catch (Exception e){
            trace.exception(status, e);
            throw e; //예외를 안던져주면 애플리케이션 코드에 영향을 주게 됨
        }
    }
```

### **_Service_**

```java
    public void orderItem(String itemId){
        TraceStatus status = null;
        try{
            status = trace.begin("OrderServiceV1.request()");
            orderRepository.save(itemId);
            trace.end(status);
        }catch (Exception e){
            trace.exception(status, e);
            throw e; //예외를 안던져주면 애플리케이션 코드에 영향을 주게 됨
        }
    }
```

### **_Repository_**

```java
    public void save(String itemid){

        TraceStatus status = null;
        try{
            status = trace.begin("OrderRepositoryV1.request()");

            if(itemid.equals("ex")){
                throw new IllegalStateException("예외 발생!");
            }

            //예외가 발생 안하면 실행
            sleep(1000);

            trace.end(status);
        }catch (Exception e){
            trace.exception(status, e);
            throw e; //예외를 안던져주면 애플리케이션 코드에 영향을 주게 됨
        }
    }
```

begin()는 계층의 메서드가 호출될 때 가장 먼저 실행되어 로그추적기의 상태값을 변경한다.

반대로 end() 또는 exception()는 예외의 발생 여부에 따라 계층의 메서드가 종료가 되기전에 호출되어 상태값을 변경한다.

이를 통해 메서드에 파라미터를 추가하여 상태값을 인자로 계속 넘겨줄 필요가 없어진 것이다.

```java
실행 결과

OrderController.request()
 [95b033fc] |-->OrderServiceV1.request()
 [95b033fc] |   |-->OrderRepositoryV1.request()
 [95b033fc] |   |<--OrderRepositoryV1.request() time=1014ms
 [95b033fc] |<--OrderServiceV1.request() time=1014ms
 [95b033fc] OrderController.request() time=1014ms
```

문제없이 잘 작동되는 것처럼 보인다.

하지만 여기에는 매우 큰 문제가 존재한다. 위는 하나의 쓰레드의 동작에 따른 결과이지만 스프링 부트의 경우에 멀티쓰레드로 동작하게 된다. FieldLogTrace 클래스는 수동으로 빈을 주입하였는데, 이 때 싱글톤 빈으로 등록하여 사용한다.

바로 이 싱글톤 빈에서 문제가 발생한다. 싱글톤 빈에는 읽기 용도로 사용해야만 한다는 것은 이제 너무 당연하다. 하지만 위에서 동기화를 위해 쓰기 용도로 필드를 하나 추가하여 사용한다.

동시에 여러개의 요청이 오게 될 경우 멀티 쓰레드로 동작이 되면서 해당 필드의 값이 모든 쓰레드에서 공유하여 사용하기 때문에 문제가 발생하게 된다. (JVM의 Heap 영역)

```java
실행 결과(2개의 요청을 동시에 보냈을 때)

 [ecb2355d] OrderController.request()
 [ecb2355d] |-->OrderServiceV1.request()
 [ecb2355d] |   |-->OrderRepositoryV1.request()
 [ecb2355d] |   |   |-->OrderController.request()
 [ecb2355d] |   |   |   |-->OrderServiceV1.request()
 [ecb2355d] |   |   |   |   |-->OrderRepositoryV1.request()
 [ecb2355d] |   |<--OrderRepositoryV1.request() time=1013ms
 [ecb2355d] |<--OrderServiceV1.request() time=1013ms
 [ecb2355d] OrderController.request() time=1013ms
 [ecb2355d] |   |   |   |   |<--OrderRepositoryV1.request() time=1014ms
 [ecb2355d] |   |   |   |<--OrderServiceV1.request() time=1015ms
 [ecb2355d] |   |   |<--OrderController.request() time=1015ms

```

요청한 클라이언트를 구분하기 위한 UUID값도 동일하게 나오고, 단계도 이상하게 중첩이 되어 로그가 기록된다.

이는 당연하게도 필드의 값을 공유해서 사용하다 보니 발생하는 문제이다.

UUID는 클라이언트의 요청마다 새롭게 만들어져서 사용하게 되는데, 요청이 동시에 여러개가 올 경우에는 처음에 만들어서 넣어둔 상태값을 이후의 요청에서 그대로 가져다가 사용하기 떄문에 새롭게 만들지 않고 다음 단계의 상태값을 넣어준다는 것이다.

근본적인 문제는 모든 쓰레드에서 동기화 필드 값을 공유해서 사용하기 때문이다.  
그럼 쓰레드마다 각기 다른 동기화 필드 값을 사용하게 만들면 해결할 수 있을것이다.

이를 해결하기 위한 방법이 바로 **_"ThreadLocal"_** 이다.

</br>

---

## **_ThreadLocal 예제_**

ThreadLocal은 스프링에서 제공하는 문법이 아니라 JAVA에서 제공하는 문법이다.

즉, 스프링에 의존적인 문법이 아니라는 장점이 존재한다.

ThreadLocal을 간단하게 설명하면 멀티쓰레드 환경에서 각각의 쓰레드가 자신의 영역에서만 사용할 수 있는 공간을 만들어 거기에 변수를 저장한다고 생각하면 된다. 당연히 쓰레드간에 서로의 공간에 접근할 수 없다.

### **_예제 코드_**

우선적으로 ThreadLocal을 사용하지 않는 경우의 예시를 보자.

```java
public class FieldService {

    private String nameStore;

    public String logic(String name){
        log.info("저장 name={} -> nameStore={}",name, nameStore);
        nameStore = name;
        sleep(1000);
        log.info("조회 nameStore={}", nameStore);
        return nameStore;
    }
}
```

해당 클래스는 nameStore라는 필드와 logic이라는 메서드가 존재한다.  
이를 2개의 쓰레드에서 동시에 실행시켜보자.

```java
    @Test
    void field(){
        log.info("main start");

        Runnable userA = () -> {
            fieldService.logic("userA");
        };
        Runnable userB = () -> {
            fieldService.logic("userB");
        };

        Thread threadA = new Thread(userA);
        threadA.setName("thread-A");
        Thread threadB = new Thread(userB);
        threadB.setName("thread-B");

        threadA.start();
        sleep(100);
        threadB.start();

        sleep(3000); //메인 쓰레드 종료 대기
    }

실행 결과
main start
[thread-A] INFO hello.advanced.trace.threadlocal.code.FieldService - 저장 name=userA -> nameStore=null
[thread-B] INFO hello.advanced.trace.threadlocal.code.FieldService - 저장 name=userB -> nameStore=userA
[thread-A] INFO hello.advanced.trace.threadlocal.code.FieldService - 조회 nameStore=userB
[thread-B] INFO hello.advanced.trace.threadlocal.code.FieldService - 조회 nameStore=userB
```

처음에 쓰레드A에서 logic()를 수행하여 첫번째 로그를 출력하고 저장한 후 1초를 대기한다. 1초를 대기하는 동안 쓰레드B가 동작되어 두번째 로그를 출력하고 저장한 후 1초를 대기한다.

이후 시간이 지남에 따라 조회 로그를 두 쓰레드가 출력하게 되는데, 쓰레드A에서 저장한 값은 userA이니 조회 결과도 userA가 나와야 하는데 조회하기 전에 쓰레드B에서 필드 값을 userB로 변경하였기 때문에 조회 결과 또한 userB가 나오게 된 것이다.

그럼 이제 이것을 ThreadLocal로 변경하여 실행해 보자.

```java
public class ThreadLocalService {

    private ThreadLocal<String> nameStore = new ThreadLocal<>();

    public String logic(String name){
        log.info("저장 name={} -> nameStore={}",name, nameStore.get());
        nameStore.set(name);
        sleep(1000);
        log.info("조회 nameStore={}", nameStore.get());
        return nameStore.get();
    }
}
```

nameStore의 필드를 ThreadLocal로 변경하였다. 이에 따라 각 쓰레드에서 nameStore를 사용하게 될 경우 자신의 영역에 별도의 공간이 마련된다.  
이후 set()에 따라 자신의 공간에 값을 저장한다. 저장된 값을 가져다가 사용하는 것은 get()이다.

이를 위 Test 코드를 수행하게 되면

```java
실행 결과
 [main] INFO hello.advanced.trace.threadlocal.FieldServiceTest - main start
 [thread-A] INFO hello.advanced.trace.threadlocal.code.ThreadLocalService - 저장 name=userA -> nameStore=null
 [thread-B] INFO hello.advanced.trace.threadlocal.code.ThreadLocalService - 저장 name=userB -> nameStore=null
 [thread-A] INFO hello.advanced.trace.threadlocal.code.ThreadLocalService - 조회 nameStore=userA
[thread-B] INFO hello.advanced.trace.threadlocal.code.ThreadLocalService - 조회 nameStore=userB
```

동일한 테스트 코드로 원하던 결과대로 동작하게 되었다.

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/205478422-1e5c5f7f-414b-4ee1-bfee-47f9ec4bd024.png" width = 70%>
  </p>

실제로 동작되는 방식은 위 이미지와 같이 쓰레드 로컬 객체로 선언된 필드는 모든 쓰레드에서 공유해서 사용하지만 객체의 get(), set() 실행로직을 통해 쓰레드마다 별도의 공간에 저장하고자 하는 값이 저장이 되는 것이다.

</br>

```
의문점
ThreadLocal의 set()를 통해 여러개의 값을 저장하고 나서 get()를 실행하고 나면 마지막으로 저장한 값을 꺼내온다.

그럼 특정한 값을 꺼내오는 방법이나 저장한 모든 값을 꺼내오는 방법 또한 따로 메서드가 정의되어 있지 않을까 하는 생각으로 ThreadLocal의 코드를 보았지만 별도로 존재하는 것 같지는 않았다.

그래서 찾아본 결과
https://stackoverflow.com/questions/2795447/is-there-no-way-to-iterate-over-or-copy-all-the-values-of-a-java-threadlocal

위와 같이 따로 직접 구현을 하든 값을 저장할 때 컬렉션 방식으로 저장을 하든 해야할 것 같다.

여러개의 ThreadLocal을 사용하게 되는 경우에는 ThreadLocalMap에 저장하게 되기 때문에 Map의 사용방법처럼 특정 객체에는 접근할 수 있는 것 같다.

이 외에도 ThreadLocal을 얕게 찾아봤는데 값을 바로 저장하는게 아닌 entity를 만들어 거기서도 key를 통해 값을 따로 저장하는 것 같으며, key는 약한 참조로 이루어져 있고, 그 위의 entitiy와 ThreadLocal(?) 간에는 강한 참조로 이루어져 있는 것 같은데, 궁금증을 해결하려다 오히려 더 복잡해지는 것 같아 우선적으로 이정도만 알고 넘어가고, 다음에 깊게 알아야 할 때에 다시 공부하자.

```

</br>

---

## **_ThreadLocal 적용_**

ThreadLocal을 어떻게 사용해야 하는지를 예제를 통해 봤으니, 이제 실제로 적용을 해보자.

```java
public class ThreadLocalLogTrace implements LogTrace{

  private ThreadLocal<TraceId> traceIdHolder = new ThreadLocal<>();

    private void syncTraceId(){
        TraceId traceId = traceIdHolder.get();
        if(traceId == null){
            traceIdHolder.set(new TraceId());
        } else {
            traceIdHolder.set(traceId.createNextId());
        }
    }

    private void releaseTraceId() {
        TraceId traceId = traceIdHolder.get();
        if(traceId.isFirstLevel()){
            traceIdHolder.remove();
        } else {
            traceIdHolder.set(traceId.createPreviousId());
        }
    }

}
```

기존의 동기화 필드를 ThreadLocal 객체로 선언하고, 제네릭 타입은 TraceId로 선언하였다.

이후 해당 필드를 사용하는 부분들을 ThreadLocal 객체를 사용하는 목적에 맞게 저장하고 가져다가 사용하면 될 뿐이다.

이후 수동으로 빈을 등록하는 부분에만 ThreadLocalLogTrace로 변경하고, 기존의 V3버전 코드를 그대로 실행만 하면 된다.

이를 실행하게 되면 아무리 여러번 호출이 되어도, 쓰레드마다 별도로 사용하기 때문에 이전의 문제점을 보완하게 되는 것이다.

</br>

---

## **_ThreadLocal 주의점_**

ThreadLocal을 사용하고 나면 반드시 remove를 통해 해당 쓰레드에서 사용하고 있던 공간을 제거해야 한다.

이유는 Tomcat이 쓰레드를 만들고 사용하는 방법 때문이다.

```
https://github.com/sksrpf1126/study/blob/main/Spring(Spring%20Boot)/%EC%8A%A4%ED%94%84%EB%A7%81%20MVC%201%ED%8E%B8%20-%20%EB%B0%B1%EC%97%94%EB%93%9C%20%EC%9B%B9%20%EA%B0%9C%EB%B0%9C%20%ED%95%B5%EC%8B%AC%20%EA%B8%B0%EC%88%A0/%EC%84%9C%EB%B8%94%EB%A6%BF%20%EC%A0%95%EB%A6%AC.md

에서 Tomcat의 쓰레드 동작원리가 정리되어 있다.
```

Tomcat은 매 요청마다 쓰레드를 만들어서 사용하는 것이 아닌 쓰레드 풀이라는 공간에 일정 수의 쓰레드를 미리 만들어서 필요할 때마다 꺼내 쓰고, 반납하는 형태로 동작하게 된다.

즉, 쓰레드 자체를 재사용한다는 것인데, 이전의 쓰레드를 사용할 때에 ThreadLocal을 사용하고 제거를 하지 않고 그대로 반납을 한 상태에서 다시 해당 쓰레드를 가져다가 사용하는 경우, ThreadLocal의 공간이 그대로 남아있게 된다.

잘못하면 이전 사용자의 정보를 다른 사용자에 제공이 될 수 있다는 것이다.

그러니 쓰레드 풀 방식으로 사용하는 경우에는 ThreadLocal을 무조건 remove를 해주어야 한다.

</br>

---

## **_다음 내용_**

다음 내용으로는 ThreadLocal을 학습하는 과정 중에 다른 수강생들이 남긴 질문들을 보고 정리해보고자 한다.

1. ThreadLocal 대신 웹 스코프(request)로는 대안책이 될 수 있는가?
2. ThreadLocal 대신 동기화 컬렉션 프레임워크로는 대안책이 될 수 있는가? (예시 : ConcurrentHashMap)
3. ThreadLocal 대신 프로토타입 빈으로는 대안책이 될 수 있는가?

위 3개의 경우를 ThreadLocal의 역할을 대신하는 코드를 직접 작성함으로써, 대안책이 될 수 있는지를 정리해보자.
