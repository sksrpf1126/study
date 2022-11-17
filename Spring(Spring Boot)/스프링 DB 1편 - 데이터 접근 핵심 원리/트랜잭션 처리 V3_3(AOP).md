# **_트랜잭션 처리 V3_3(AOP)_**

해당 내용은 스프링 DB 1편 - 데이터 접근 핵심 원리(김영한님)의 강의를 보고 정리한 내용입니다.

---

V3_2에서는 서비스단에서 비즈니스 로직 뿐만 아니라 트랜잭션 관련된 로직(TransacionTemplate)이 같이 존재하였다.

서비스 단에는 순수한 비즈니스 로직만을 사용하기 위해서는 어떻게 해야할까?

바로 스프링의 AOP기술을 활용하면 된다. 트랜잭션을 처리하는 로직을 AOP에 직접 작성해서 처리를 해도 되지만, 트랜잭션은 서버에서 필수로 어디서든 사용하기 때문에 스프링은 트랜잭션만을 처리하는 AOP, 즉 트랜잭션 AOP 기술을 지원해준다.

해당 기술을 통해 트랜잭션 중복 코드를 제거해 보자.

</br>

---

## **_트랜잭션 AOP_**

젹용하기 전에 트랜잭션 AOP에 대해서 알아보자.

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/202109879-a881bbd2-1912-4b49-a782-63ef04616292.png" width = 70%>
</p>

위는 트랜잭션 AOP를 적용하기 전, 그러니까 Service V3_2 버전의 흐름이다.  
서비스 단에 비즈니스 로직뿐만 아니라 트랜잭션 관련 로직도 들어가 있는 것을 확인할 수 있다.

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/202109873-1b7dd2ef-dcec-4f2f-a440-4e85c36f75f1.png" width = 70%>
</p>

위는 트랜잭션 AOP를 적용하고 나서의 흐름이다.  
서비스단에는 순수한 비즈니스 로직만이 존재하고, 서비스 앞에 트랜잭션 프록시라는 것이 생기면서 클라이언트는 실제로는 서비스를 직접적으로 호출하는 것이 아닌 앞의 트랜잭션 프록시 코드를 호출하게 된다. 그리고 프록시 내부 코드에는 서비스의 비즈니스 로직을 호출하는 부분이 들어가게 되는 것이다.

이를 통해 서비스단에는 비즈니스 로직만이 존재하게 되는 것이다.

이러한 트랜잭션 AOP를 적용하기 위해서는 정말 간단하다.  
서비스의 실행할 메서드(또는 클래스)에 @Transactional 애노테이션만 붙이고 비즈니스 로직만을 메서드에 담아두면 위 방식으로 동작이 된다.

</br>

### **_트랜잭션 프록시?_**

트랜잭션 프록시는 무엇이며, 어떻게 동작이 되는 것일까  
위에서 @Transacional 애노테이션이 존재하면 스프링은 트랜잭션 프록시를 생성한다.

생성하는 방법은 이전에 @Configuraion에 @Bean을 통해 수동 빈을 등록할 때 싱글톤으로 만들어지는 원리를 공부할 때에 CGLIB 라이브러리 기술이 활용되는 것을 보았다.

바로 해당 라이브러리 기술이 트랜잭션 프록시를 생성할 때에도 사용이 되는 것이다.(테스트 코드에서 확인할 예정)

CGLIB 라이브러리를 간단하게 설명하면 바이트 조작을 하여 코드를 수정하는 것인데, 이를 자세하게 설명하면

1. 빈으로 등록하고자 하는 클래스를 상속받는 자식 클래스를 생성한다. (getClass를 해보면 뒤에 CGLIB가 붙은것을 확인할 수 있음)

2. 상속받는 자식 클래스에서 메서드들을 오버라이딩을 하는데, 사용하는 입장에서는 빈으로 등록하였던 클래스의 메서드를 호출하겠지만 실제로는 동적바인딩을 통한 CGLIB에 의해 정의된 자식클래스의 오버라이딩된 메서드가 호출이 된다.

위 원리가 트랜잭션 프록시 코드에도 동일하게 적용이 되는 것이다.

트랜잭션 프록시 코드를 간단하게 살펴보자면

```java
public class TransactionProxy {

  private MemberService target;

  public void logic() {
    //트랜잭션 시작
    TransactionStatus status = transactionManager.getTransaction(..);
    try {
    //실제 대상 호출
    target.logic();
    transactionManager.commit(status); //성공시 커밋
    } catch (Exception e) {
    transactionManager.rollback(status); //실패시 롤백
    throw new IllegalStateException(e);
    }
  }
}
```

위에서는 상속과 오버라이딩은 제거하고 순수하게 로직을 통한 동작원리를 살펴보는 것이다.

@Transactional 애노테이션을 보고 CGLIB가 동작되어 위와 같은 코드가 만들어지고 실행이 된다. 쉽게 말해서 트랜잭션을 처리하는 로직을 알아서 만들어주고 적절한 떄에 비즈니스 로직을 호출하게 된다.

</br>

---

## **_Service V3_3_**

```java
public class MemberServiceV3_3 {

    private final MemberRepositoryV3 memberRepository;

    public MemberServiceV3_3(MemberRepositoryV3 memberRepository) {
        this.memberRepository = memberRepository;
    }

    //@Transactional에 의해 해당 메서드가 호출 될 때 프록시 코드가 만들어져서 트랜잭션 처리 해줌
    //클래스에 붙이면 public 메서드가 모두 적용
    @Transactional
    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        bizLogic(fromId, toId, money);
    }
}
```

변경된 부분만 보면 기존의 TransactionTemplate을 제거하였으며, 비즈니스 로직을 호출하는 메서드 또한 트랜잭션을 처리하는 로직이 전부 없어졌다.

@Transactional 애노테이션만 붙이면 끝이다.

</br>

---

## **_Test_**

```java
@Slf4j
@SpringBootTest
class MemberServiceV3_3Test {

  ...

    @Autowired
    private MemberRepositoryV3 memberRepository;

    @Autowired
    private MemberServiceV3_3 memberService;

    @TestConfiguration
    static class TestConfig {

        @Bean
        DataSource dataSource(){
            return new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        }

        @Bean
        PlatformTransactionManager transactionManager() {
            return new DataSourceTransactionManager(dataSource());
        }

        @Bean
        MemberRepositoryV3 memberRepositoryV3() {
            return new MemberRepositoryV3(dataSource());
        }

        @Bean
        MemberServiceV3_3 memberServiceV3_3() {
            return new MemberServiceV3_3(memberRepositoryV3());
        }
    }

    ...
}
```

변경된 부분만을 보자.

기존에 @BeforEach로 DI를 진행한 후에 테스트 코드를 실행하였지만 @Transactional을 테스트하기 위해서는 해당 애노테이션이 스프링에 의존적이기 때문에 @SpringBootTest를 통해 스프링을 띄워야 한다. 그러다 보니 굳이 직접적으로 DI를 할 필요 없이 스프링의 방식으로 빈을 등록하고 사용하면 되기 떄문에 @BeforEach방식은 제거한 것이다.

이후 테스트 코드를 동작하면 정상적으로 통과가 된다.

</br>

그러면 위에서 설명한 CGLIB의 원리가 적용되었는지 확인을 해보자.

```java
    @Test
    void AopCheck() {
        log.info("memberService class={}", memberService.getClass());
        log.info("memberRepository class={}", memberRepository.getClass());
        Assertions.assertThat(AopUtils.isAopProxy(memberService)).isTrue();
        Assertions.assertThat(AopUtils.isAopProxy(memberRepository)).isFalse();
    }

  실행결과
  memberService class=class hello.jdbc.service.MemberServiceV3_3$$EnhancerBySpringCGLIB$$853bc83d
  memberRepository class=class hello.jdbc.repository.MemberRepositoryV3
```

memberService에 CGLIB가 붙은 것을 통해 확인을 할 수 있다.

이후 AOP 프록시의 여부에 대한 테스트도 진행해보면 정상적으로 통과가 되는 것을 확인할 수 있다.

</br>

---

## **_트랜잭션 AOP 최종 동작원리_**

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/202109886-41965f52-7a36-43d4-b374-199fe1605f3c.png" width = 900%>
</p>

트랜잭션 AOP를 적용하고 나서의 동작원리이다.

이전에는 클라이언트의 요청이 서비스를 직접 호출하였다면, 트랜잭션 AOP를 통해 만들어진 프록시에 의해서 서비스가 아닌 프록시가 호출되어진다.

이후 트랜잭션을 시작하고 종료하기 까지는 선택한 DataSource와 TransactionManager 방식으로 동작하게 된다.

</br>

---

## **_정리_**

트랜잭션 AOP를 통해 이제 서비스 단에는 비즈니스 로직만을 작성하면 되며, 트랜잭션 처리 로직은 CGLIB에 의해 자동으로 만들어지고 실행이 된다.

@Transactional을 통해 트랜잭션을 처리하는 방법을 **_"선언적 트랜잭션 관리"_** 라고 한다.

그리고 트랜잭션템플릿, 트랜잭션매니저 등을 직접 코드로 작성하여 트랜잭션을 처리하는 방법을 **_"프로그래밍 방식의 트랜잭션 관리"_** 라고 한다.

실무에서는 99%가까이 선언적 트랜잭션 관리 방법을 사용한다고 한다. 하지만 선언적 트랜잭션 관리 방법은 결국 직접 작성하였던 프로그래밍 방식의 트랜잭션 관리를 자동으로 스프링이 처리를 해주는 것이다.

알고 쓰는거와 모르고 쓰는거에는 큰 차이가 존재한다.
