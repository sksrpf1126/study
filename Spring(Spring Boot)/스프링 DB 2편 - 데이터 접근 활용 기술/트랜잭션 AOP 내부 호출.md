# **_트랜잭션 AOP 내부 호출_**

해당 내용은 스프링 DB 2편 - 데이터 접근 활용 기술(김영한님)의 강의를 보고 정리한 내용입니다.

---

@Transactional 애노테이션이 메서드든 클래스에든 하나라도 존재한다면 해당 클래스(원본 클래스라 명칭)를 상속받는 프록시 객체를 만들어서 원본 클래스 인스턴스를 빈으로 등록하는 것이 아닌 프록시 객체를 빈으로 등록하여 사용하게 된다.

이를 통하여 @Transactional이 붙어있는 메서드를 호출하게 될 경우 트랜잭션을 시작하는 로직을 수행하고, 이후에 원본 클래스에서 작성한 비즈니스 로직을 호출하여 실행한다. 마지막으로 트랜잭션을 종료한다.

@Transactional이 안붙은 메서드의 경우에는 트랜잭션을 처리하지 않고 바로 원본 클래스의 메서드를 호출하게 된다.

여기서 만약에 @Transactional이 안붙은 메서드 내부에서 @Transactional이 붙은 메서드를 직접 호출(내부 호출)한다면 트랜잭션은 처리가 될까?

</br>

---

## **_내부 호출 테스트_**

```java
    static class CallService {

        public void external(){
            log.info("call external");
            printTxInfo();
            internal();
        }

        @Transactional
        public void internal(){
            log.info("call internal");
            printTxInfo();
        }

        private void printTxInfo() {
            boolean txActive = TransactionSynchronizationManager.isActualTransactionActive();
            log.info("tx active={}", txActive);
        }
    }
```

코드는 위와 같이 @Transactional을 선언한 internal 메서드와 선언하지 않은 external 메서드가 있으며, external 메서드에서는 internal 메서드를 직접 호출하는 로직이 존재한다.

위 구조에서 external 메서드를 호출해보자.

```java
실행 결과:
call external
tx active = false //트랜잭션 적용 여부
call internal
tx active = false //트랜잭션 적용 여부
```

실행 결과를 확인해보면 내부에서 직접 호출하는 경우에는 트랜잭션 처리가 되지 않는 것을 볼 수 있다.

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/204076848-13d6aedc-3134-4af7-980f-51c1a34fdb31.png" width = 70%>
  </p>

적용되지 않는 이유는 위의 이미지와 같이 동작하기 때문이다.

@Transactional이 하나라도 존재하기 때문에 프록시 객체가 생성되어 빈으로 등록이 된다.

다음으로 프록시 객체의 external 메서드를 호출하게 되는데, @Transactional이 존재하지 않기 때문에 그냥 바로 원본 인스턴스의 메서드를 호출한다.

이후, 원본 인스턴스의 external메서드에서는 내부에 internal 메서드를 호출하게 되는데, 이때에는 this.internal이므로 프록시 객체의 internal이 아닌 원본 인스턴스의 internal을 호출이 되기 때문에 트랜잭션을 적용하지 않게 되는 것이다.

실무에서 많이 발생하는 문제이기 때문에 면접에서 이에 대해 질문을 많이 한다고 한다. (ex) @Transactional을 사용할 때 주의해야 할 점 or 스프링에서 트랜잭션을 처리할 떄 주의해야 할 점)

</br>

---

## **_실무에서 위 문제를 해결하는 방법_**

실무에서 위 방식으로 처리해야 되는 로직이 있을 때 해결하는 방법은 @Transactional 즉, 트랜잭션을 처리해야하는 메서드를 같은 클래스에 두지 않고 별도의 클래스로 따로 선언하여 외부 호출로 돌리는 것이다.

```java
    static class InternalService {
        @Transactional
        public void internal(){
            log.info("call internal");
            printTxInfo();
        }
    }
```

```java
    static class CallService {

        private final InternalService internalService;

        @Transactional
        public void external(){
            log.info("call external");
            printTxInfo();
            internalService.internal();
        }
    }
```

internal 메서드를 같은 클래스가 아닌 다른 클래스에 두어서 internalService.internal()로 외부 호출을 한다.

```java
실행 결과:
 call external
 tx active=false
 Getting transaction for [hello.springtx.apply.InternalCallV2Test$InternalService.internal]
 call internal
 tx active=true
 Completing transaction for [hello.springtx.apply.InternalCallV2Test$InternalService.internal]
```

위와 같이 internal 메서드가 호출이 될 때 정상적으로 트랜잭션이 적용되는 것을 확인할 수 있다.

</br>

---

## **_@Transacional은 public에만_**

트랜잭션 AOP는 메서드가 public일 때에만 정상적으로 적용이 되고 나머지 접근 제어자는 적용이 안된다.

내부 호출의 개념이랑 관련되어 있는 것은 아니고, 스프링에서 일부로 막아뒀다고 한다.

그 이유는 @Transacional 이 붙은 메서드는 보통 트랜잭션을 처리해야하는 비즈니스 로직을 수행하는 메서드일테고, 그러면 여러 다른 경로(패키지)에서 호출을 하게 될 터이니 public을 거의 필수로 선언해야 하며, 클래스 레벨에 @Transactional에 public 이외의 다른 접근 제어자로 선언된 메서드들에게는 적용되지 않게 하기 위함이다.
