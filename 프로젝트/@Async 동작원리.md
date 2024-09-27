# ***@Async 동작원리***

해당 내용은 [테이크 잇(TakeEat)](https://github.com/pickpong/takeeat) 프로젝트를 진행하며, 정리한 내용입니다.

해당 글에서는 이메일 인증 로직을 구현 중에 @Asnyc가 적용되지 않아 @Asnyc의 동작방식과 해결한 과정을 ***간략히*** 정리하였습니다.  

---

### ***동기? 비동기?***

@Async의 동작방식을 정리하기 전에 우선 알아야하는 개념은 동기와 비동기의 개념입니다.  

동기(Synchronous) 와 비동기(Asynchronous)는 아래의 이미지로 쉽게 접근해서 이해할 수 있습니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/39ccb342-7d03-483d-93f1-7a6f211fcf63" width = 100%></p> 

위 이미지는 동기와 비동기에서 4개의 업무를 진행했을 때를 비교하여 보여주는 이미지입니다.  

동기(왼쪽)의 경우 하나의 업무가 끝나면 다음 업무를 작업합니다.  
그래서 4개의 작업시간을 합쳐서 총 45초가 소요가 됩니다.  

반대로 비동기(오른쪽)의 경우 4개의 업무를 동시에 처리하여 20초가 소요가 됩니다.  

비동기 방식은 하나씩 처리하는 방식이 아니고 동시에 처리하게 되어 처리속도가 굉장히 빠릅니다.  

동기와 비동기를 정말 간단하게 설명하면 위 설명이 끝입니다. 하지만 여기서 궁금한 것은 과연 어떻게 동작이 되면 오른쪽과 같이 여러 업무를 동시에 처리하는 걸까? 라는 의문이 생깁니다.  

#### ***비동기의 동작방식***

비동기의 동작방식을 이해하기 위해서는 Thread라는 것을 알고, 싱글 쓰레드와 멀티 쓰레드의 개념을 알아야 이해가 됩니다.    
해당 내용은 [Thread 정리](https://github.com/sksrpf1126/study/blob/main/java/java%20%EA%B8%B0%EC%B4%88%EB%AC%B8%EB%B2%95/Thread.md) 에 정리를 해놓지만 정말 간단하게 설명을 해보자면 비동기는 ***하나의 쓰레드를 기준(싱글 쓰레드)*** 으로 여러 작업을 정말 순식간에 번갈아가면서 처리하는 것입니다.    

예를 들어 1번 작업 10%하고, 2번 작업으로 넘어가서 10%.. 이런식인데 너무 빨라서 사람의 눈에는 동시에 처리되는 것처럼 보이는 것입니다.    

근데 이러면 동기 방식이랑 성능 차이가 나면 안됩니다. 오히려 성능은 비동기 방식이 더 오래걸려야 정상입니다. 왜냐하면 작업을 변경하는 과정(컨텍스트 스위칭)에 의해 오히려 더 시간이 소요가 되기 때문입니다.  

사실 정확히 말하면 위 내용은 단 하나의 쓰레드 즉, 싱글 쓰레드를 기준으로 설명하는 것입니다.  

동기는 하나의 쓰레드로 작업을 하나씩 쳐낸다면, 비동기는 여러 쓰레드 즉, 멀티 쓰레드 방식으로 여러 작업들을 각각의 쓰레드에서 처리하는 것입니다.  

더 정확히 말하면 CPU의 코어 수가 많으면 많을수록 더 많은 작업을 동시에 처리할 수 있습니다.  
아래는 인텔 코어 i5-14500 CPU 의 성능을 가져온 것입니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/1f7832b5-e64a-4a07-8d42-c25b7aa7ddf7" width = 100%></p> 

하드웨어 쪽을 정확히 알지는 못하지만 정말 쉽게 얘기하면 코어 하나가 하나의 작업을 처리한다고 보면 이해가 쉽습니다. 즉, 코어 수가 많으면 많을수록 동시에 처리할 수 있는 작업 수가 많아진다는 것입니다.  

### ***모든 작업을 비동기로 처리하도록 구현하면 되겠네요?***

Tomcat 즉, WAS는 Thread Pool이라는 곳이 존재합니다.  
사용자의 요청마다 쓰레드를 만들어서 작업하고, 작업을 완료하면 쓰레드를 제거하고... 이와 같이 매 요청마다 쓰레드를 만들고 제거하고 이러면 성능이 매우 나빠집니다.  

그래서 Thread Pool 이라는 곳에 처음에 정해진 수만큼만 쓰레드를 만들어두고 빌려주고 반납받는 식으로 동작하게 됩니다.  

근데 비동기는 각각의 작업을 각각의 쓰레드 즉, 멀티 쓰레드 방식으로 동작하게 됩니다.  

만약 연속적인 작업이 존재하는데, 해당 작업들이 정말 사소한 작업인데도 불구하고 비동기로 처리하게 된다면 전체적으로 쓰레드 수는 부족하게 되어 문제가 발생할 것이라고 개인적으로 생각합니다.  

그래서 보통 인증메일을 발송하는 경우에 대표적으로 비동기로 처리합니다.  
동기 처리를 하게 되면 인증메일 발송이 완료가 될 때까지 계속 기다리게 되어 사용자 또한 해당 시간 동안 기다리게 됩니다.  

하지만 비동기로 처리하게 되면 인증메일 작업을 따로 처리하도록 하고 나머지 작업을 이어나가서 사용자에게 빠르게 응답하도록 구현할 수 있습니다.  

---

### 그래서 @Async는?

@Async는 단 한마디로 정의하면, 해당 어노테이션이 걸려있는 메서드를 비동기로 처리하라는 의미입니다.  

동기와 비동기를 이해했다면 @Async는 금방 이해가 됩니다.  

하지만 주의해야할 점이 존재합니다.  

### @Async 사용 시 주의할 점
1. method 접근지정자 private 사용 불가
2. self-invocation(자가 호출) 불가, 즉 inner method는 사용 불가

1번의 경우는 @Async를 거는 메서드의 접근 지정자를 private으로 둘 수 없다는 것입니다.  

그러면 왜 private은 사용이 불가능할까요?  

***@Async는 Spring AOP에 의해서 프록시 방식으로 동작*** 되기 때문입니다.  

프록시(프록시 객체)는 Spring에서 정말 많이 사용됩니다.  
예를 들어 JPA에서 지연 로딩에서도 사용이 되고, @Transactional에서도 사용이 됩니다.  

대표적으로 사용되는 부분은 AOP입니다.(AOP까지 정리하면 내용이 너무 길어져서 패스하겠습니다..)  

또 하나는 해당 글의 주제인 @Async 또한 사용이 되죠.  

그러면 프록시 객체가 뭐길래 private은 사용이 될 수 없다는 것일까요?  

프록시 객체는 원본 객체를 상속받아서 만들어지며, 만들어 질 때 내부적으로 로직이 추가됩니다.  

@Transcational이 걸려있는 메서드의 경우에는 프록시 객체가 만들어지면서 원본 메서드의 로직의 실행 전과 후에 로직이 추가됩니다.  

예시로는 다음과 같습니다.  

```java
public void 프록시객체메서드() {
  //트랜잭션 시작
  TransactionStatus status = transactionManager.getTransaction(..);
  try {
  //실제 대상 호출
  원본객체.메서드();
  transactionManager.commit(status); //성공시 커밋
  } catch (Exception e) {
  transactionManager.rollback(status); //실패시 롤백
  throw new IllegalStateException(e);
  }
}
```
위와 같이 원본객체의 실행할 메서드 위 아래에 트랜잭션을 처리하는 로직이 추가되는 것입니다.  

그러면 @Async 또한 동일하게 원본 객체의 메서드와 비동기를 처리하는 로직이 같이 존재하는 프록시 객체가 탄생하는 것입니다.  

근데 여기서 원본객체의 메서드가 private이라면??  
프록시 객체가 원본 객체의 메서드를 호출할 수 없게 되버립니다. 그래서 private 접근 지정자는 불가능하다는 것입니다.  

### self-invocation(자가 호출) 불가, 즉 inner method는 사용 불가

그럼 위 말은 무슨 의미일까요?  

쉽게 말해 @Async, @Transactional과 같이 프록시 객체가 만들어질 때 로직이 추가되는 친구들을 원본 객체내에서 직접 호출해서 쓰는 경우입니다.  

아래의 로직으로 테스트해보겠습니다.  

```java
public class AsyncService {

  @Async
  public void methodA() {
      log.info("[{}] methodA Call", this.getCurrentThreadName());
      this.methodB();
  }

  @Async
  public void methodB() {
      log.info("[{}] methodB Call", this.getCurrentThreadName());
  }

  private String getCurrentThreadName() {
      return Thread.currentThread().getName();
  }

}
```

위와 같이 methodA와 methodB는 AsyncService라는 원본 클래스(객체) 내에 같이 존재하는 메서드이며, @Async가 각각 걸려있습니다.  
그리고 methodA에서 methodB를 직접 호출하고 있습니다.  

각각 정상적으로 비동기 처리가 되면, methodA가 호출될 때와 methodB를 호출될 때 다른 쓰레드에서 각각 동작이 되어야 합니다. 
이를 확인하기 위해서 각각의 메서드에 동작되고 있는 쓰레드의 이름을 log로 찍어봤습니다.  

그리고 methodA를 호출하게 되면 아래와 같이 동작됩니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/3f5459ee-c64b-41ac-bbac-b608d6c0d908" width = 70%></p> 

@Async가 각각 있음에도 불구하고 단 하나의 쓰레드에서 동기방식으로 동작하게 됩니다.  

이와 같이 methodA에서 this.methodB를 직접적으로 호출하게 되면 프록시 객체에서 추가된 로직이 존재하는 메서드를 호출하는 것이 아니라 원본 객체의 메서드를 이어서 호출하기 때문에 정상적으로 실행이 되지 않습니다.  

이와 같은 문제를 해결하기 위해서는 다른 클래스에 각각 분리해서 호출해야 합니다.  

변경한 코드는 다음과 같습니다.  

```java
@Service
@Slf4j
public class AsyncSubService {

    @Async
    public void methodB() {
        log.info("[{}] methodB Call", Thread.currentThread().getName());
    }

}
```

methodB를 AsyncSubService 클래스에 선언을 하였습니다.  

```java
public class AsyncService {

  @Async
  public void methodA() {
      log.info("[{}] methodA Call", this.getCurrentThreadName());
      asyncSubService.methodB();
  }

  private String getCurrentThreadName() {
      return Thread.currentThread().getName();
  }

}
```

그리고 AsyncService에서는 methodA에서 asyncSubService의 methodB를 호출하도록 합니다.  

그리고 동일하게 methodA를 호출하게 된다면??

<p align ="center"><img src="https://github.com/user-attachments/assets/3b858866-04ee-4455-8e79-64e81afeccbe" width = 70%></p> 

위와 같이 정상적으로 각각 쓰레드를 할당받아서 처리하는 것을 확인할 수 있습니다.  

---

## ***이메일 인증에서 @Async 동작되지 않은 이유***

위 내용을 정리하게 된 이유는 프로젝트의 이메일 인증 로직 구현중에 @Async가 정상적으로 동작이 되지 않아서입니다.  

본론부터 말하면, 주의할 점의 2번 부분입니다.  

로직은 다음과 같습니다.  

```java
@Transactional
public void authenticationEmail(String email) {

    if(memberRepository.existsByEmail(email)) {
        throw new BaseException(ErrorCode.MEMBER_EXISTS);
    }

    String authCode = this.createCode();

    emailAuthRepository.save(EmailAuth.builder()
            .email(email)
            .authCode(authCode)
            .build());

    this.sendEmail(email, authCode, "[TakeEat] 회원가입 이메일 인증");
}

@Async
public void sendEmail(String email, String authCode, String subject) {
    SimpleMailMessage mailMessage = new SimpleMailMessage();
    mailMessage.setTo(email);
    mailMessage.setSubject(subject);
    mailMessage.setText("인증 코드는 " + authCode + " 입니다.");

    log.info("{} 로 인증코드를 방송 했습니다. code : {}", email, authCode);

    javaMailSender.send(mailMessage);
}
```

위 authenticationEmail 메서드의 마지막 로직을 보면 this.sendEmail이라는 메서드를 호출하몋 해당 메서드는 @Async가 걸려있음에도 불구하고, 같은 클래스에서 직접적으로 호출하고 있습니다.  

그래서 결국은 비동기가 정상적으로 실행이 되지 않았으며. 해당 로직을 그대로 호출한다면?  

<p align ="center"><img src="https://github.com/user-attachments/assets/c5a5f1a7-606a-44cd-b505-51ed8fcb0948" width = 100%></p> 

자세히 봐야할 부분은 걸린 시간으로, 4.84초가 소요된 것을 볼 수 있습니다.  

그러면 비동기가 정상적으로 실행이 되도록 변경하게 로직을 수정해보겠습니다.  

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class AsyncService {

    private final JavaMailSender javaMailSender;

    @Async
    public void sendEmail(String email, String authCode, String subject) {
        SimpleMailMessage mailMessage = new SimpleMailMessage();
        mailMessage.setTo(email);
        mailMessage.setSubject(subject);
        mailMessage.setText("인증 코드는 " + authCode + " 입니다.");

        log.info("{} 로 인증코드를 방송 했습니다. code : {}", email, authCode);

        javaMailSender.send(mailMessage);
    }

}
```
우선 @Async가 걸려있는 sendEmail 메서드를 별도의 AsyncService 클래스로 옮겼습니다.  

그리고 해당 메서드를 호출하도록 변경합니다.  

```java
    @Transactional
    public void authenticationEmail(String email) {

        if(memberRepository.existsByEmail(email)) {
            throw new BaseException(ErrorCode.MEMBER_EXISTS);
        }

        String authCode = this.createCode();

        emailAuthRepository.save(EmailAuth.builder()
                .email(email)
                .authCode(authCode)
                .build());

        asyncService.sendEmail(email, authCode, "[TakeEat] 회원가입 이메일 인증");
    }
```
this.sendEmail => asyncSerivce.sendEmaill로 변경하여 직적접으로 호출하는 것이 아닌 프록시가 적용된 sendEmail 메서드를 호출합니다.  

그리고 호출하게 되면  

<p align ="center"><img src="https://github.com/user-attachments/assets/335354bd-da34-45f8-8137-bd581b5c5c99" width = 100%></p>  

<p align ="center"><img src="https://github.com/user-attachments/assets/e6dde16d-246c-4af1-afde-55e8686e31fa" width = 100%></p>

<p align ="center"><img src="https://github.com/user-attachments/assets/00f8a623-335c-45fa-97e3-34a7a5f56296" width = 100%></p>

위와 같이 3번을 테스트하였으며, 차례대로 소요시간은 562ms, 640ms, 614ms 가 걸렸으며, 3개의 평균을 내면 605ms정도 소요되는 것을 확인할 수 있었습니다.  

이전에 소요되었던 4.84s에서 비동기를 정상적으로 적용 후에 605ms로
약 7.94배의 속도차이가 나는 것을 확인했습니다.  

### 동기, 비동기 방식의 이메일 인증 화면 비교

<table>
	<tbody>
		<tr>
			<th></th>
		</tr>
		<tr>
			<td><img width="100%" src="https://github.com/user-attachments/assets/3a17e01a-4cf4-471f-a6ea-81cf82e669ef" alt="랜딩 페이지"/></td>
		</tr>		
    </tbody>

</table>

왼쪽은 비동기가 적용된 이메일 인증 발송 화면이고, 오른쪽은 비동기가 적용되지 않은 이메일 인증 발송 화면입니다.  

비동기가 적용된 이메일 인증의 경우 0.61초가 소요되었고, 적용되지 않은 경우에는 3.85초가 소요되었습니다.  
