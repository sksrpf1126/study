# **_ThreadLocal -2 (대안책)_**

해당 내용은 스프링 핵심 원리 - 고급편 (김영한님)의 강의를 보고 정리한 내용입니다.

---

ThreadLocal은 쓰레드 별로 별도의 공간에 값을 저장한다는 개념이다.  
그래서 이전에 로그 추적기를 직접 만들어서 사용할 때에 상태값을 쓰레드별로 저장하도록 하여 여러 문제점을 개선하였다.

그럼 위 ThreadLocal의 역할을 다른 방식으로 구현할 수 있지 않을까?

강의에서 질문내용을 정리해보니 3가지 정도로 추려낼 수 있었다.

1. ThreadLocal 대신 웹 스코프(request)로는 대안책이 될 수 있는가?
2. ThreadLocal 대신 동기화 컬렉션 프레임워크로는 대안책이 될 수 있는가? (예시 : ConcurrentHashMap)
3. ThreadLocal 대신 프로토타입 빈으로는 대안책이 될 수 있는가?

위 3가지의 경우로 나누어서 ThreadLocal의 **_역할_** 을 대신할 수 있는지를 판단해보자.

</br>

---

## **_웹 스코프 활용_**

질문내용 출처: https://www.inflearn.com/questions/340172/%EC%8A%A4%EB%A0%88%EB%93%9C-%EB%A1%9C%EC%BB%AC%EA%B3%BC-request-%EC%8A%A4%EC%BD%94%ED%94%84

웹 스코프(request)는 HTTP 요청마다 별도의 빈을 만들어서 사용하게 된다. 클라이언트의 HTTP 요청이 올 때 쓰레드가 생기며, 해당 쓰레드에서 웹 스코프를 가지는 빈이 생성되어서 사용이 되는 것이다.

그럼 각각의 쓰레드마다 별도의 빈을 만들어지고, 해당 빈에서 필드를 정의해서 사용하면, 이 필드들은 결국 모든 쓰레드에서 공유하는 필드가 아닌 해당 쓰레드에서만 사용이 되는 필드로 제한할 수 있을 것이다.

이를 코드로 구현하여 실행해보자.

```java
@Slf4j
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class WebScopeLogTrace implements LogTrace{

    private static final String START_PREFIX = "-->";
    private static final String COMPLETE_PREFIX = "<--";
    private static final String EX_PREFIX = "<X-";

    private TraceId traceIdHolder; //traceId 동기화

    @Override
    public TraceStatus begin(String message) {
        syncTraceId();
        TraceId traceId = traceIdHolder;
        long startTimeMs = System.currentTimeMillis();
        log.info("[{}] {}{}", traceId.getId(), addSpace(START_PREFIX, traceId.getLevel()), message);
        return new TraceStatus(traceId, startTimeMs, message);
    }

    private void syncTraceId(){
        if(traceIdHolder == null){
            traceIdHolder = new TraceId();
        } else {
            traceIdHolder = traceIdHolder.createNextId();
        }
    }


    ...
}
```

코드가 더 길지만 동기화 필드의 방식이랑 웹 스코프로만 변경할 뿐 나머지 메서드들의 로직들은 크게 변경된 부분이 없다.

주의 깊게 봐야할 부분은 동기화를 위한 클래스의 필드가 ThreadLocal이 아니라, ThreadLocal의 이전 방법으로 되돌아 갔다는 것이다.

private TraceId traceIdHolder 로 클래스의 하나의 필드일 뿐이다.

웹 스코프가 아니라면 동시에 여러 요청이 오면 모든 쓰레드가 같은 필드를 사용하기 때문에 문제가 발생할 것이지만 웹 스코프의 방식이기 때문에 의도한 대로라면 여러 요청이 와도 쓰레드마다 웹 스코프 빈이 새롭게 만들어지기 때문에 별도의 필드값을 사용하게 될 것이다.

이제 의도한 방향대로 동작이 되는지를 확인해보자.

</br>

```java
 [nio-8080-exec-3] h.a.trace.logtrace.WebScopeLogTrace      : [6ea70928] OrderController.request()
 [nio-8080-exec-3] h.a.trace.logtrace.WebScopeLogTrace      : [6ea70928] |-->OrderServiceV1.request()
 [nio-8080-exec-3] h.a.trace.logtrace.WebScopeLogTrace      : [6ea70928] |   |-->OrderRepositoryV1.request()
 [nio-8080-exec-1] h.a.trace.logtrace.WebScopeLogTrace      : [df4bc708] OrderController.request()
 [nio-8080-exec-1] h.a.trace.logtrace.WebScopeLogTrace      : [df4bc708] |-->OrderServiceV1.request()
 [nio-8080-exec-1] h.a.trace.logtrace.WebScopeLogTrace      : [df4bc708] |   |-->OrderRepositoryV1.request()
 [nio-8080-exec-3] h.a.trace.logtrace.WebScopeLogTrace      : [6ea70928] |   |<--OrderRepositoryV1.request() time=1002ms
 [nio-8080-exec-3] h.a.trace.logtrace.WebScopeLogTrace      : [6ea70928] |<--OrderServiceV1.request() time=1002ms
 [nio-8080-exec-3] h.a.trace.logtrace.WebScopeLogTrace      : [6ea70928] OrderController.request() time=1005ms
 [nio-8080-exec-2] h.a.trace.logtrace.WebScopeLogTrace      : [d71e9f31] OrderController.request()
 [nio-8080-exec-2] h.a.trace.logtrace.WebScopeLogTrace      : [d71e9f31] |-->OrderServiceV1.request()
 [nio-8080-exec-2] h.a.trace.logtrace.WebScopeLogTrace      : [d71e9f31] |   |-->OrderRepositoryV1.request()
 [nio-8080-exec-4] h.a.trace.logtrace.WebScopeLogTrace      : [deba3850] OrderController.request()
 [nio-8080-exec-4] h.a.trace.logtrace.WebScopeLogTrace      : [deba3850] |-->OrderServiceV1.request()
 [nio-8080-exec-4] h.a.trace.logtrace.WebScopeLogTrace      : [deba3850] |   |-->OrderRepositoryV1.request()
 [nio-8080-exec-1] h.a.trace.logtrace.WebScopeLogTrace      : [df4bc708] |   |<--OrderRepositoryV1.request() time=1011ms
 [nio-8080-exec-1] h.a.trace.logtrace.WebScopeLogTrace      : [df4bc708] |<--OrderServiceV1.request() time=1011ms
 [nio-8080-exec-1] h.a.trace.logtrace.WebScopeLogTrace      : [df4bc708] OrderController.request() time=1011ms
 [nio-8080-exec-5] h.a.trace.logtrace.WebScopeLogTrace      : [0538b17b] OrderController.request()
 [nio-8080-exec-5] h.a.trace.logtrace.WebScopeLogTrace      : [0538b17b] |-->OrderServiceV1.request()
 [nio-8080-exec-5] h.a.trace.logtrace.WebScopeLogTrace      : [0538b17b] |   |-->OrderRepositoryV1.request()
 [nio-8080-exec-2] h.a.trace.logtrace.WebScopeLogTrace      : [d71e9f31] |   |<--OrderRepositoryV1.request() time=1007ms
 [nio-8080-exec-2] h.a.trace.logtrace.WebScopeLogTrace      : [d71e9f31] |<--OrderServiceV1.request() time=1007ms
 [nio-8080-exec-2] h.a.trace.logtrace.WebScopeLogTrace      : [d71e9f31] OrderController.request() time=1007ms
 [nio-8080-exec-4] h.a.trace.logtrace.WebScopeLogTrace      : [deba3850] |   |<--OrderRepositoryV1.request() time=1003ms
 [nio-8080-exec-4] h.a.trace.logtrace.WebScopeLogTrace      : [deba3850] |<--OrderServiceV1.request() time=1005ms
 [nio-8080-exec-4] h.a.trace.logtrace.WebScopeLogTrace      : [deba3850] OrderController.request() time=1005ms
 [nio-8080-exec-5] h.a.trace.logtrace.WebScopeLogTrace      : [0538b17b] |   |<--OrderRepositoryV1.request() time=1013ms
 [nio-8080-exec-5] h.a.trace.logtrace.WebScopeLogTrace      : [0538b17b] |<--OrderServiceV1.request() time=1013ms
 [nio-8080-exec-5] h.a.trace.logtrace.WebScopeLogTrace      : [0538b17b] OrderController.request() time=1014ms

```

매우 짧은 시간안에 5번의 요청을 보냈다. 쓰레드 또한 총 5개가 동작되는 것을 확인할 수 잇으며, UUID값이 매 요청마다 새로운 값이 할당이 되는 것을 확인할 수 있다. 또한 UUID값으로 구분해서 로그를 보면 ThreadLocal과 동일하게 동작되는 것을 확인할 수 있다.

이를 통해 ThreadLocal의 대안책으로 웹 스코프를 활용할 수 있다는 것이다.

그렇다면, ThreadLocal 대신에 웹 스코프로 구현해도 별 차이가 없다는 것일까?  
사실 둘에게는 차이점이 존재한다.

웹 스코프는 HTTP요청이 와야한다는 조건과 스프링에 의존적인 방식이라는 것이다. 하지만 ThreadLocal은 스프링이 아닌 자바에서 제공해주는 문법이므로, 특정 라이브러리나 프레임워크에 의존적이지 않다는 것이다.

</br>

---

## **_동기화 컬렉션 프레임워크_**

질문내용 출처 : https://www.inflearn.com/questions/347336/threadlocal-%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%9D%B4%EC%8A%88-arraylist-hashmap-hashset

다음으로는 ThreadLocal 대신에 ConcurrentHashMap을 사용해 보는 것이다.  
ConcurrentHashMap은 강의에서도 여러번 언급이 된 멀티쓰레드 환경에서의 동기화를 보장해주는 클래스이다.

어찌보면 ThreadLocal의 쓰레드 동기화와 같은 의미로 느껴질 수 있다. 하지만 둘의 동기화는 다른 문제를 해결한다는 것이다.

ThreadLocal에서는 쓰레드마다 별도의 공간에 값을 저장하고 사용하여, **_"쓰레드 동기화"_** 를 해결한다.  
동기화 컬렉션의 경우에는 동기화 컬렉션으로 선언된 필드를 여러 쓰레드가 처리할려고 할 때에 같은 요소를 처리할려고 하면 락(Lock) 을 통해 먼저 처리할려고 하는 쓰레드를 제외한 나머지 쓰레드는 대기를 해야한다.

이후 처리가 완료된 쓰레드는 락을 반납하고, 다른 쓰레드가 접근할 수 있도록 하는 것으로 **_"쓰레드 동기화"_** 를 해결하는 것이다.

정리하면 쓰레드마다 ThreadLocal 객체를 통해 별도의 "값"을 접근해서 사용하므로, 쓰레드 간에 ThreadLocal로 처리하는 값들은 서로의 쓰레드와는 무관하다는 것이다.

하지만 동기화 컬렉션은 해당 필드에 저장된 값들(요소들)은 여러 쓰레드가 동시에 값을 처리하지 못하게 막을 뿐 결국 모든 쓰레드가 해당 필드를 공유해서 사용한다는 것이다.

```
+심화 내용+
ConcurrentHashMap에 10개의 요소가 저장되어 있을 경우 각각의 요소마다 락으로 제어한다는 것(부분 잠금)이다.
그러니 같은 요소를 처리하는 쓰레드가 아닐 경우에는 병렬적으로 처리한다고 한다.

이를 통해 좀 더 빠르게 처리할 수 있다고 한다.

출처 : https://cornswrold.tistory.com/209
```

테스트 코드를 통해서 무엇이 다른지를 자세히 확인해보자.

</br>

```java
@Slf4j
public class ConcurrentHashMapService {

//    private ThreadLocal<String> nameStore = new ThreadLocal<>();
    private ConcurrentHashMap<String, String> nameStore = new ConcurrentHashMap<>();

    public String logic(String name){
        log.info("저장 name={} -> nameStore={}",name, nameStore.get(name));
        sleep(1000);
        nameStore.put(name, name);
        log.info("조회 nameStore={}", nameStore.get(name));
        return nameStore.get(name);
    }

    public void NameStorePrint(String name){
        sleep(3000);
        log.info("nameStore.get() = {}", nameStore.get(name));
    }
}
```

기존의 ThreadLocal의 테스트를 위한 코드를 그대로 복사해와서 nameStore를 ThreadLocal이 아닌 ConcurrentHashMap으로 변경하였다.

이를 테스트하는 테스트 코드를 보자.

</br>

```java
public class ConcurrentHashMapTest {

    private ConcurrentHashMapService fieldService = new ConcurrentHashMapService();

    @Test
    void field(){
        log.info("main start");

        Runnable userA = () -> {
            fieldService.logic("userA");
            fieldService.NameStorePrint("userA");
        };
        Runnable userB = () -> {
            fieldService.logic("userB");
            fieldService.NameStorePrint("userB");
        };

        Thread threadA = new Thread(userA);
        threadA.setName("thread-A");
        Thread threadB = new Thread(userB);
        threadB.setName("thread-B");

        threadA.start();
        sleep(100);
        threadB.start();

        sleep(5000);
    }
}
```

쓰레드가 실행이 될 때 logic 메서드와 NameStorePrint 메서드를 실행한다.  
우선 logic 메서드에서는 저장 관련 로그를 남기고 1초를 대기한 후 nameStore에 값을 넣은다음에 조회 로그를 남긴다.

다음으로 NameStorePrint 메서드는 3초를 대기한 후 nameStore의 값을 로그로 남긴다.

테스트 코드에서는 두개의 쓰레드를 실행하며, 둘의 실행속도 차이는 0.1초의 텀을 두었다.

```java
실행 결과
[main] INFO hello.advanced.trace.threadlocal.ConcurrentHashMapTest - main start
[thread-A] INFO hello.advanced.trace.threadlocal.code.ConcurrentHashMapService - 저장 name=userA -> nameStore=null
[thread-B] INFO hello.advanced.trace.threadlocal.code.ConcurrentHashMapService - 저장 name=userB -> nameStore=null
[thread-A] INFO hello.advanced.trace.threadlocal.code.ConcurrentHashMapService - 조회 nameStore=userA
[thread-B] INFO hello.advanced.trace.threadlocal.code.ConcurrentHashMapService - 조회 nameStore=userB
[thread-A] INFO hello.advanced.trace.threadlocal.code.ConcurrentHashMapService - nameStore.get() = userA
[thread-B] INFO hello.advanced.trace.threadlocal.code.ConcurrentHashMapService - nameStore.get() = userB
```

실행결과를 보면, 쓰레드A와 쓰레드B 둘 다 각각 다른 값을 저장하고 이후 key값으로 get()를 실행해서 조회해 보면 값을 잘 가져온다.

결과만 보면 ThreadLocal처럼 각기 다른 값을 사용하기 때문에 대안책이 되는 것으로 보이지만, 테스트 코드를 살짝 바꿔서 다시 보자.

```java
  @Test
  void field(){
      log.info("main start");

      Runnable userA = () -> {
          fieldService.logic("userA");
          //fieldService.NameStorePrint("userA");
      };
      Runnable userB = () -> {
          //fieldService.logic("userB");
          fieldService.NameStorePrint("userA");
      };

      Thread threadA = new Thread(userA);
      threadA.setName("thread-A");
      Thread threadB = new Thread(userB);
      threadB.setName("thread-B");

      threadA.start();
      sleep(100);
      threadB.start();

      sleep(5000);
  }
```

쓰레드A에서는 userA의 값을 저장하고 저장하고 난 후에 바로 조회를 해보고 종료한다.  
쓰레드B에서는 아무런 값도 저장하지 않고, userA라는 값이 nameStore에 저장되어 있는지만을 확인한다.

만약 정말로 ThreadLocal과 같은 **_"쓰레드 동기화"_** 문제를 해결한다면, 쓰레드A에서 저장한 값을 쓰레드B에서 조회해도 값은 존재하지 않아야 한다.

실행결과를 보자.

</br>

```java
실행결과
[main] INFO hello.advanced.trace.threadlocal.ConcurrentHashMapTest - main start
[thread-A] INFO hello.advanced.trace.threadlocal.code.ConcurrentHashMapService - 저장 name=userA -> nameStore=null
[thread-A] INFO hello.advanced.trace.threadlocal.code.ConcurrentHashMapService - 조회 nameStore=userA
[thread-B] INFO hello.advanced.trace.threadlocal.code.ConcurrentHashMapService - nameStore.get() = userA
```

쓰레드B에서 쓰레드A가 저장한 userA값을 조회했는데, 다른 쓰레드에서 저장한 값임에도 출력해서 가져온다.  
이를 통해 동기화 컬렉션은 쓰레드마다 별도의 필드로 사용하는게 아니라 모든 쓰레드가 공유해서 사용하는 필드라는 것이다.

결국 ThreadLocal의 대안책으로 될 수 없다는 것이다.

</br>

---

## **_프로토타입 빈_**

질문내용 출처: https://www.inflearn.com/questions/519888/thread-local-%EA%B3%BC-prototype-bean-%EB%AC%B8%EC%9D%98

마지막으로 프로토타입 빈이 ThreadLocal의 대안책이 될 수 있을까에 대한 내용이다.

```java
@Getter @Setter
public class PrototypeStore {

    private String name;

    @PostConstruct
    public void init(){
        System.out.println("PrototypeBean.init");
    }
}
```

우선 프로토타입 빈으로 등록할 클래스이다.  
프로토타입 빈 방식으로 잘 동작되는지 확인하기 위해서 @PostConstruct를 통해 빈이 생성되고 주입된 후에 로그를 남긴다.

```java
@Slf4j
public class PrototypeService {

    @Autowired
    private Provider<PrototypeStore> prototypeBeanProvider;

    public String logic(String name){
        PrototypeStore nameStore = prototypeBeanProvider.get();
        log.info("저장 name={} -> nameStore={}, nameStore class={}",name, nameStore.getName(), nameStore);
        nameStore.setName(name);
        sleep(1000);
        log.info("조회 nameStore={}, nameStore class={}", nameStore.getName(), nameStore);
        return nameStore.getName();
    }
}
```

로그 출력 부분에 nameStore 값 자체를 볼 수 있도록 추가하였으며, 싱글톤 빈에 각기 다른 프로토타입 빈을 주입하기 위해서 Provider 문법을 활용하였다.

</br>

```java
@Slf4j
@SpringBootTest
public class PrototypeServiceTest {

    @Autowired
    private PrototypeService fieldService;

    @TestConfiguration
    static class Config {

        @Bean
        public PrototypeService prototypeService(){
            return new PrototypeService();
        }

        @Scope("prototype")
        @Bean
        public PrototypeStore prototypeStore(){
            return new PrototypeStore();
        }
    }

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

        sleep(5000);
    }
}
```

프로토타입 빈을 테스트하기 위해서 @SpringBootTest를 추가하였고, @TestConfiguration을 통해 수동으로 빈을 주입하여서 사용한다.

그리고 PrototypeStore 빈은 프로토타입 빈으로 등록한다.

이제 실행해보자.

```java
실행결과
[           main] h.a.t.threadlocal.PrototypeServiceTest   : main start
[       thread-A] h.a.t.threadlocal.code.PrototypeStore    : PrototypeBean.init
[       thread-A] h.a.t.threadlocal.code.PrototypeService  : 저장 name=userA -> nameStore=null, nameStore class=hello.advanced.trace.threadlocal.code.PrototypeStore@58d189f2
[       thread-B] h.a.t.threadlocal.code.PrototypeStore    : PrototypeBean.init
[       thread-B] h.a.t.threadlocal.code.PrototypeService  : 저장 name=userB -> nameStore=null, nameStore class=hello.advanced.trace.threadlocal.code.PrototypeStore@3404f825
[       thread-A] h.a.t.threadlocal.code.PrototypeService  : 조회 nameStore=userA, nameStore class=hello.advanced.trace.threadlocal.code.PrototypeStore@58d189f2
[       thread-B] h.a.t.threadlocal.code.PrototypeService  : 조회 nameStore=userB, nameStore class=hello.advanced.trace.threadlocal.code.PrototypeStore@3404f825
```

쓰레드마다 PrototypeBean.init 로그가 남는 것을 보면 프로토타입 빈으로 주입이 된 것을 확인할 수 있으며, nameStore 객체의 주소값을 통해 두 쓰레드의 nameStore가 다른 객체라는 것을 또 확인할 수 있다.

이러면 쓰레드마다 별도의 프로토타입 빈을 사용하니까 ThreadLocal의 대안책이 될 수 있는 것 같지만 logic메서드의 동작방식을 보면 다르다는 것을 알 수 있다.

```java
    public String logic(String name){
        PrototypeStore nameStore = prototypeBeanProvider.get();
        log.info("저장 name={} -> nameStore={}, nameStore class={}",name, nameStore.getName(), nameStore);
        nameStore.setName(name);
        sleep(1000);
        log.info("조회 nameStore={}, nameStore class={}", nameStore.getName(), nameStore);
        return nameStore.getName();
    }
```

PrototypeStore nameStore = prototypeBeanProvider.get()에서 logic메서드가 실행이 될 때마다 프로토타입 빈을 생성 및 주입을 하여 사용하게 된다. 하지만 이 코드에서 nameStore는 지역변수의 개념으로써 사용이 된다는 것이다.

지역변수는 JVM의 Runtime Data Area의 어디 영역에서 저장되고 사용이 되는지를 떠올려 보자.  
바로 쓰레드마다 별도의 영역을 할당받는 stack 영역에 logic 메서드가 호출되어 올라갈 때에 선언되서 사용이 되다가 logic 메서드가 종료가 될 때 지역변수 또한 사라지게 된다.

여기서 다시 ThreadLocal이 어떻게 사용이 되는지를 다시 생각해보자.
ThreadLocal 객체 자체는 모든 쓰레드에서 접근 가능한 필드라는 것.  
이후 내부의 값만이 쓰레드마다 별도로 저장되고 사용을 한다는 것.
마지막으로 내부에서 사용되는 값은 해당 필드만 사용이 가능하다면 어떠한 메서드에서든지 꺼내서 사용할 수 있다는 것이다.

그러면 프로토타입 빈을 지역변수에 저장하지 말고 필드에 저장해서 사용하면 되는 것 아닌가?

```java
    private PrototypeStore nameStore; //동기화를 위한 필드

    public String logic(String name){
        nameStore = prototypeBeanProvider.get();
        log.info("저장 name={} -> nameStore={}, nameStore class={}",name, nameStore.getName(), nameStore);
        nameStore.setName(name);
        sleep(1000);
        log.info("조회 nameStore={}, nameStore class={}", nameStore.getName(), nameStore);
        return nameStore.getName();
    }
```

PrototypeService 클래스에 위와 같이 필드로 빼서 프로토타입 빈을 필드에 저장한다.  
하지만 이 순간 해당 필드는 모든 쓰레드에서 공유해서 사용되는 필드가 된 것이고, 여러 요청이 동시에 들어오면 해당 필드의 값이 이상하게 변경될 것이다.

즉, 프로토타입 빈 만으로는 ThreadLocal의 대안책이 될 수 없다는 것이다.

</br>

---

## **_ThreadLocal + 프로토타입 빈_**

물론 ThreadLocal에 프로토타입 빈을 생성해서 저장하고 사용하면 해결이 된다. 하지만 이 순간 프로토타입 빈만으로는 대안책이 될 수 없다는 걸 다시 한 번 입증한다는 것이다.

```java
@Slf4j
public class PrototypeService {

    @Autowired
    private Provider<PrototypeStore> prototypeBeanProvider;

//    private PrototypeStore nameStore; //동기화를 위한 필드
    private ThreadLocal<PrototypeStore> nameStore = new ThreadLocal<>();

    public String logic(String name){
        PrototypeStore prototypeStore = prototypeBeanProvider.get();
        log.info("저장 name={} -> nameStore={}",name, nameStore.get() == null ? null : nameStore.get().getName());
        prototypeStore.setName(name);
        nameStore.set(prototypeStore);
        sleep(1000);
        log.info("조회 nameStore={}", nameStore.get().getName());
        return nameStore.get().getName();
    }
}
```

위와 같이 logic 메서드가 실행이 될 때마다 프로토타입 빈의 생성 및 주입이 이루어지며, 이를 ThreadLocal 객체에 저장하고 사용한다.  
물론 객체를 값으로 저장하였기 때문에 NPE를 주의해야 한다...

</br>

---

## **_정리_**

처음에 적은 질문 내용의 결과

1. ThreadLocal 대신 웹 스코프(request)로는 대안책이 될 수 있는가? O (HTTP 요청 + Spring 한정)
2. ThreadLocal 대신 동기화 컬렉션 프레임워크로는 대안책이 될 수 있는가? X (ConcurrentHashMap)
3. ThreadLocal 대신 프로토타입 빈으로는 대안책이 될 수 있는가? X
