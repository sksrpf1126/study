# **_트랜잭션 전파 REQUIRES_NEW_**

해당 내용은 스프링 DB 2편 - 데이터 접근 활용 기술(김영한님)의 강의를 보고 정리한 내용입니다.

---

이전에는 Required(트랜잭션 전파 기본 옵션)를 기준으로 트랜잭션 전파에 대해 공부했고, 이번에는 그 다음으로 많이 쓰이는 REQUIRES_NEW 옵션을 알아보자.

REQUIRES_NEW 옵션의 경우에는 외부 트랜잭션과 내부 트랜잭션을 분리해서 각각 별도의 물리 트랜잭션을 사용하는 방법이다. 즉, 커밋과 롤백이 자신의 트랜잭션에만 영향을 준다는 것이다.

옵션은 외부 트랜잭션(처음 시작하는 트랜잭션)이 아닌 내부 트랜잭션에 추가해야 한다.

</br>

---

## **_REQUIRES_NEW 원리_**

```java
    @Test
    void inner_rollback_requires_new(){
        log.info("외부 트랜잭션 시작");
        TransactionStatus outer = txManager.getTransaction(new DefaultTransactionAttribute());
        log.info("outer.isNewTransaction()={}",outer.isNewTransaction()); //true

        log.info("내부 트랜잭션 시작");
        DefaultTransactionAttribute definition = new DefaultTransactionAttribute();
        definition.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
        TransactionStatus inner = txManager.getTransaction(definition);
        log.info("inner.isNewTransaction()={}",inner.isNewTransaction()); //true -> REQUIRES_NEW 속성에 의하여

        log.info("내부 트랜잭션 롤백");
        txManager.rollback(inner);

        log.info("외부 트랜잭션 커밋");
        txManager.commit(outer);
    }
```

위에서 getTransaction() 즉, 트랜잭션을 시작하기전에 definition이라는 객체에 설정정보를 담아서 인자로 전달한다.  
설정정보에는 TransactionDefinition.PROPAGATION_REQUIRES_NEW 를 통해서 내부 트랜잭션에 옵션을 추가하였다.

외부 트랜잭션에서는 commit을 하지만 내부 트랜잭션에서는 rollback을 하는 경우이다.

이제 위 코드의 실행결과를 보자.

</br>

```java
 외부 트랜잭션 시작
 Creating new transaction with name [null]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
 Acquired Connection [HikariProxyConnection@118280482 wrapping conn0: url=jdbc:h2:mem:8a205c19-0a76-474a-b949-79ee83282d8e user=SA] for JDBC transaction
 Switching JDBC Connection [HikariProxyConnection@118280482 wrapping conn0: url=jdbc:h2:mem:8a205c19-0a76-474a-b949-79ee83282d8e user=SA] to manual commit
 outer.isNewTransaction()=true
 Suspending current transaction, creating new transaction with name [null]
 Acquired Connection [HikariProxyConnection@1899091560 wrapping conn1: url=jdbc:h2:mem:8a205c19-0a76-474a-b949-79ee83282d8e user=SA] for JDBC transaction
 Switching JDBC Connection [HikariProxyConnection@1899091560 wrapping conn1: url=jdbc:h2:mem:8a205c19-0a76-474a-b949-79ee83282d8e user=SA] to manual commit
 inner.isNewTransaction()=true
 Initiating transaction rollback
 Rolling back JDBC transaction on Connection [HikariProxyConnection@1899091560 wrapping conn1: url=jdbc:h2:mem:8a205c19-0a76-474a-b949-79ee83282d8e user=SA]
 Releasing JDBC Connection [HikariProxyConnection@1899091560 wrapping conn1: url=jdbc:h2:mem:8a205c19-0a76-474a-b949-79ee83282d8e user=SA] after transaction
 Resuming suspended transaction after completion of inner transaction
 외부 트랜잭션 커밋
 Initiating transaction commit
 Committing JDBC transaction on Connection [HikariProxyConnection@118280482 wrapping conn0: url=jdbc:h2:mem:8a205c19-0a76-474a-b949-79ee83282d8e user=SA]
 Releasing JDBC Connection [HikariProxyConnection@118280482 wrapping conn0: url=jdbc:h2:mem:8a205c19-0a76-474a-b949-79ee83282d8e user=SA] after transaction
```

우선 커넥션 객체가 어떠한 방식으로 동작되는지를 보면 외부 트랜잭션은 conn0 객체를 랩핑해서 사용하며 주소는 @118280482으로 할당이 되어있다.

다음으로 내부 트랜잭션에서의 커넥션 객체는 conn1 객체를 랩핑해서 사용하며 주소는 @1899091560으로 할당이 되어있다.

이를 정리하면 이전의 Required 옵션에서는 **_"참여"_** 의 형태로 하나의 커넥션 객체에서 동작이 되었지만, REQUIRES_NEW 옵션의 경우에는 각기 다른 커넥션 객체를 할당받아서 사용하는 것을 확인할 수 있다.

트랜잭션이 처음 시작되는 것인지의 여부를 확인하는 isNewTransaction()에서 또한 둘다 true값이라는 것을 확인할 수 있다.

이후 내부 트랜잭션이 rollback을 하고서는 conn1을 반납하고 종료한다. 이후 외부 트랜잭션 또한 commit을 하고서는 conn0을 반납하고 종료한다.

이렇게 외부, 내부 트랜잭션은 각각 별도의 물리 트랜잭션이 할당되어 동작되는 것을 볼 수 있다.

</br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/204734235-07db5e62-2e0a-4aaa-9bbc-305cf04a97e6.png" width = 70%>
  </p>

자세한 동작은 위의 이미지와 같이 동작이 된다.

각강의 커넥션 객체가 DataSource를 통해서 별도로 할당이 되어 트랜잭션 동기화 매니저가 관리한다. 이후 commit or rollback을 할 때에는 자신에게 할당된 커넥션 객체를 사용하는 것으로써 두 트랜잭션은 서로 영향을 끼치지 않는 것이다.

</br>

---

## **_이 외의 여러 옵션들_**

트랜잭션 전파 옵션에는 기본 옵션인 Required와 이번에 알아본 REQUIRES_NEW 옵션 이외에도 다양한 옵션이 존재한다.

SUPPORT, NOT_SUPPORT, MANDATORY, NEVER, NESTED 가 존재하지만 실무에서는 위 2개의 옵션 이외에는 사용하는 경우가 거의 없다고 한다.
