# ***RDB와 Redis 이메일 인증***

해당 내용은 [테이크 잇(Take Eat)](https://github.com/pickpong/takeeat) 프로젝트를 진행하며, 정리한 내용입니다.

해당 글에서는 기존에 RDB를 통해 이메일을 인증하던 방식에서 Redis를 사용한 이메일 인증 방식으로 변경하는 내용을 정리하였습니다.  

---

### ***프로젝트에서 이메일 인증을 선택한 이유***

해당 프로젝트에서 회원가입을 하기 위해서는 이메일을 인증해야 합니다.  
해당 프로젝트에 더 어울리는 인증 방식은 당연히 핸드폰 인증인 것임을 알고 있지만, 핸드폰 인증을 위한 별도의 핸드폰 번호와 문자 송신에 의한 요금 부담에 의하여 팀원들과 회의를 통하여 핸드폰 인증에서 이메일 인증으로 수단을 변경하였습니다.  

또한, 이메일 인증과 휴대폰 인증은 수단의 방법 차이일 뿐 인증 절차의 로직은 크게 다를 것 없어 이메일 인증으로도 충분히 많은 것을 배울 수 있다고 생각하여 이메일 인증으로 선택하였습니다.  

---

## ***RDB를 통한 이메일 인증 방식***

현재 프로젝트에서는 RDB(MySQL)를 통하여 이메일 인증 로직이 구현되어 있습니다.  

현재 이메일 인증과 회원의 ERD 구조입니다.  
두 테이블의 연관관계는 존재하지 않습니다.

<p align ="center"><img src="https://github.com/user-attachments/assets/5b93dd9e-6773-41d0-8e47-3e50557d4cc2" width = 100%></p> 

#### ***이메일 인증 엔티티***

```java
@Entity
@Getter
@EntityListeners(value = AuditingEntityListener.class)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class EmailAuth {
    private static final Long EXPIRE_TIME = 5L;

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "email_auth_id")
    private Long emailAuthId;

    private String email;

    private String authCode;

    @CreatedDate
    private LocalDateTime createdTime;

    @Builder
    public EmailAuth(String email, String authCode) {
        this.email = email;
        this.authCode = authCode;
    }

    /**
     * 인증코드 만료 여부 체크
     * @return 만료(true), 아니라면(false)
     */
    public boolean expireCheck() {
        Long minutes = Duration.between(this.createdTime.toLocalTime(), LocalDateTime.now().toLocalTime()).toMinutes();

        return minutes >= EXPIRE_TIME ? true : false;
    }

}
```
이메일 인증에 사용되는 엔티티이며, PK값으로 사용되는 emailAuthId, 인증 요청을 보낸 email 그리고 해당 이메일로 보내게 되는 인증코드(authCode) 필드가 들어가게 됩니다.  

expireCheck 메서드는 사용자가 입력한 인증코드가 만료가 되었는지 판별하는 메서드로 EXPIRE_TIME에 할당된 5라는 값을 통해서 5분으로 만료시간을 정했습니다.  

다음으로 이러한 구조에서 실제로 호출되고 실행되는 로직을 보겠습니다.  

#### ***이메일 인증요청 컨트롤러 메서드***

```java
    @PostMapping("/email-send")
    @ResponseBody
    public String sendAuthCode(@RequestParam("email") String email) {
        emailService.authenticationEmail(email);
        return "인증 코드를 발송했습니다.";
    }
```

MemberController에 선언되어 있는 메서드로, 사용자가 member/email-send 경로로 사용자가 입력한 이메일을 파라미터로 담아 전달하게 되면, emailService에 존재하는 authenticationEmail 메서드가 실행이 됩니다.  

#### ***이메일 인증요청 서비스 메서드***

그러면 다음으로 실행되는 emailService 클래스의 authenticationEmail 메서드 로직을 보겠습니다.  

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

사용자가 입력한 이메일의 값을 파라미터로 받아서 사용하며, if문에 존재하는 memberRepository.existsByEmail(email) 메서드를 통해서 해당 이메일로 이미 가입이 되어 있는 이메일인지 판단합니다.  

가입하지 않았을 때에만 this.createCode 메서드를 호출하며, 해당 메서드의 로직은 다음과 같습니다.  

```java
private String createCode() {
    int length = 6;
    try {
        Random random = SecureRandom.getInstanceStrong();
        StringBuilder builder = new StringBuilder();
        for (int i = 0; i < length; i++) {
            builder.append(random.nextInt(10));
        }
        return builder.toString();
    } catch (NoSuchAlgorithmException e) {
        throw new BaseException(ErrorCode.NO_SUCH_AUTH_CODE);
    }
}
```

쉽게 6자리의 랜덤문자열을 생성하여 반환하는 메서드입니다.  

6자리의 랜덤문자열을 반환하여  

```java
emailAuthRepository.save(EmailAuth.builder()
        .email(email)
        .authCode(authCode)
        .build());
```

사용자가 입력한 email과 authCode값을 통해서 EmailAuth 엔티티를 하나 생성하여 RDB에 저장하게 됩니다.  

마지막으로 ***asyncService.sendEmail(email, authCode, "[TakeEat] 회원가입 이메일 인증");*** 메서드를 호출하며, 해당 메서드의 로직은 다음과 같습니다.  

```java
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

파라미터로 사용자가 입력한 이메일과, 발송할 인증 코드, 그리고 이메일의 제목을 인자로 받습니다. (이메일의 제목은 상수값으로 빼서 관리하는게 좋다고 생각이 드네요)  

전달받은 이메일의 제목으로 해당 이메일로 발송이 되며, 발송될 때 인증코드를 내용에 담아서 보내게 됩니다.  

다음으로, 해당 메서드는 @Async를 통해 비동기로 동작되는 메서드입니다.  
동기로 설정되어 있을 경우 파라미터로 전달된 이메일로 메일이 전송이 될 때까지(또는 전송에 실패가 될 때까지) 다음 로직이 실행이 되지 않습니다. 결국 사용자는 완료가 될 때까지 응답을 받을 수 없습니다.  

이메일이 발송될 때까지는 서버와 네트워크 상태에 따라 다르겠지만 일반적으로 최소 3초 이상은 걸리게 되는데, 해당 시간 동안 사용자는 강제로 기다리게 됩니다.  

그렇기에 @Async를 통해 해당 메서드는 비동기로 실행되도록 해놓았습니다.  
해당 내용은 [@Async 동작원리](https://github.com/sksrpf1126/study/blob/main/%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/%40Async%20%EB%8F%99%EC%9E%91%EC%9B%90%EB%A6%AC.md) 해당 글에 정리해놨습니다.  

위 부분까지는 사용자의 이메일에 인증코드를 발송하고, 해당 인증코드를 MySQL에 저장하는 코드입니다.  

### ***사용자가 입력한 인증코드 검증***

다음으로는 사용자가 입력한 인증코드와 MySQL에 저장되어 있는 인증코드를 비교하여 검증하는 메서드입니다.  

```java
    @Transactional(readOnly = true)
    public void validateAuthCode(String email, String authCode, BindingResult bindingResult) {
        EmailAuth findEmailAuth = emailAuthRepository.findTop1ByEmailOrderByCreatedTimeDesc(email).orElse(null);

        if(findEmailAuth == null) {
            bindingResult.rejectValue("authCode", "required", "해당 이메일로 인증코드가 발송되지 않았습니다.");
        }else if(!findEmailAuth.getAuthCode().equals(authCode)) {
            bindingResult.rejectValue("authCode", "required", "인증코드를 다시 확인해 주세요.");
        }else if(findEmailAuth.expireCheck()) {
            bindingResult.rejectValue("authCode", "required", "인증코드 유효시간이 지났습니다.");
        }

    }
```

파라미터로 사용자의 이메일과 사용자가 입력한 인증코드, 그리고 예외메시지를 화면에 전달하기 위한 bindingResult를 전달받습니다.  

핵심은 findTop1ByEmailOrderByCreatedTimeDesc 메서드이며, MySQL의 EMAIL_AUTH 테이블에 해당 이메일로 발송되어 저장된 레코드들 중에 가장 최근의 레코드 1건을 가져오는 로직입니다.  

가져와서 findEmailAuth가 null이거나, MySQL에서 가져온 인증코드의 값과 사용자가 입력한 인증코드가 다르거나, 유효시간이 지났을 경우에는 bindingResult 객체에 예외정보(유효성 검증 실패 정보)를 담아 사용자에게 해당 메시지를 보여주며 회원가입은 되지 않습니다.  

</br>

### 이메일 인증에 RDB를 사용하는 것이 최선의 방법일까?

사용자에게 발송한 인증코드를 어딘가에 저장을 해두어야 이후에 사용자가 입력한 인증코드와 비교할 때 사용할 수 있습니다.  

그래서 저의 경우에는 다른 데이터들과 동일하게 인증코드 또한 MySQL에 테이블을 하나 만들어서 저장해서 사용하기로 했습니다. 하지만 해당 방식으로 하다보니 단점이 존재했습니다.  

</br>

#### `단점 1. 사용자가 인증코드를 발송 받을 때마다 RDB의 테이블에 insert 쿼리가 발생하며, 100번을 요청한다면 그 수만큼 데이터가 쌓인다.`

</br>

<p align ="center"><img src="https://github.com/user-attachments/assets/e9085a97-96b0-4c3d-a4f6-5eec64aca087" width = 70%></p> 

위와 같이 하나의 이메일에 무수히 많은 데이터가 쌓인 것을 확인할 수 있습니다.  
만약 사용자가 악의적으로 수많은 요청을 한다면 그만큼 데이터를 저장하게 됩니다.  

</br>

#### `단점 2. 인증 코드를 검증하기 위해 테이블에 접근해 해당 사용자의 이메일로 select쿼리를 날릴 때 추후 성능 이슈가 발생할 수 있다.`  


#### 무슨 성능 이슈?

무수히 많은 데이터들 중에서 특정 이메일에 해당하는 데이터를 가져와야 합니다. 그러면 인증 코드 발송 요청이 들어올 때마다 데이터가 들어가는 테이블에서 특정 이메일을 찾는 SELECT 쿼리를 날려야 합니다.  

그러면 여기서 FULL SCAN으로 모든 데이터를 뒤져가며 해당 이메일에 해당하는 데이터를 가져오고 그 중에서도 가장 마지막에 INSERT된 데이터를 조회해야 한다면?? 그렇다면 데이터가 많아질수록 FULL SCAN할 데이터 수가 많아지며, 성능 이슈가 발생합니다.  

그래서 보통 많은 데이터가 들어있는 테이블에서 조회 성능을 향상시키기 위해 인덱스 도입을 고민하게 됩니다.  
그러면 검색 조건에 해당되는 이메일에 인덱스를 걸어야 합니다. 하지만, 인덱스는 만능이 아닙니다. 인덱스를 도입하게 되면 테이블에 대해 B Tree 알고리즘 방식이 사용되는 트리구조를 만들게 됩니다.  

당연히 테이블에 데이터가 삽입, 삭제, 갱신이 될 때마다 새로운 트리구조를 만들어야 하죠. 즉, 변동이 많은 테이블의 경우에는 인덱스 도입이 오히려 악영향을 끼칩니다.  

또한, 인덱스를 도입할 때 해당 컬럼의 카디널리티가 높은가, 낮은가를 판단해서 카디널리티가 높은 즉, 중복도가 낮은 컬럼에 인덱스를 도입해야 합니다. 하지만 이메일의 경우 중복도가 매우 높습니다.  

결국 이러한 2가지 이유에 의해 해당 테이블의 이메일 컬럼에 인덱스를 도입하는 것은 오히려 성능에 악영향을 끼치게 됩니다.  

</br>

#### `단점 3. RDB는 관계형 데이터베이스이다. 하지만 이메일 인증 코드를 저장하는 테이블은 그 누구와도 관계가 없다.`  

물론, 단점이라고 뽑기는 힘들지만 그렇다고 해당 테이블은 RDB를 쓰는 이유 중 하나를 가지지 못한다는 것입니다.  
다시 말해 굳이 RDB 방식으로 할 필요도 없다는 것입니다.  


### ***생각해본 해결 방법들***

- `검색 조건에 이메일만이 아닌 인증코드와 데이터가 들어간 시간을 활용한다.`  

유효 시간이 5분이므로, 테이블에서 데이터를 조회할 때 created_time의 값이 요청한 시간의 5분 전부터 현재시간까지라는 검색 조건을 포함시키고, 또한 사용자가 입력한 인증코드가 존재하는가 존재하지 않는가도 검색 조건에 포함시킬 수 있습니다.  

이러면 훨씬 더 적은 범위부터 데이터를 찾아갈 수 있습니다.  
다음으로 사용자의 요청이 적은 시간대에 유효시간이 지난 데이터들을 삭제되게끔 설정해 두는 것입니다.  
이러면 데이터가 쌓이는 문제도 해결할 수 있습니다.  

</br>

- `Redis를 도입한다.`  

방법을 고민하고 찾던 도중에 이메일 인증에 대해 Redis를 도입하는 분들이 많은 것을 확인할 수 있었습니다.  
그러면 왜 Redis를 사용하는 것일까요?  

우선 Redis는 데이터를 저장할 때 저장시간을 지정하여, 시간이 지나면 자동으로 삭제되게 할 수 있습니다. 즉, 위에서 말한 1번 단점이 해결됩니다.  

그리고 Redis는 Key-Value 형식으로 데이터를 저장하게 됩니다. 즉, Key값으로 검색할 때 매우 빠른 속도로 데이터를 찾을 수 있습니다.  

일반적으로 Redis는 RDB보다 데이터 쓰기 작업이나, 조회 작업에 대해 속도가 월등히 빠릅니다. 이게 가능한 이유는 Redis는 하드디스크나 SSD에 데이터를 저장하는 것이 아니라 메모리에 저장하기 때문입니다. 그렇기에 접근 속도가 RDB보다 빠릅니다.  

즉, 2번 단점이 해결됩니다.  

그리고 연관관계도 없는 테이블이기에 Redis를 도입해도 기존의 테이블 구조에 영향이 가지 않습니다.  

`그래서 기존의 RDB를 사용하는 로직과 Redis의 성능을 비교한 후에 판단하기로 하였습니다.`

## ***Redis 로직 추가***

성능을 비교하기 전에 Redis 관련 로직을 추가합니다.  

### Redis Set,Get 메서드

```java
@Service
@RequiredArgsConstructor
public class RedisService {
    private final RedisTemplate<String, String> redisTemplate;

    public void setAuthCode(String email, String code){
        ValueOperations<String, String> valOperations = redisTemplate.opsForValue();
        //만료기간 5분
        valOperations.set(email,code,300, TimeUnit.SECONDS);
    }

    public String getAuthCode(String email){
        ValueOperations<String, String> valOperations = redisTemplate.opsForValue();
        return valOperations.get(email);
    }

}
```
setAuthCode 메서드는 Redis에 인증코드를 저장하는 로직이며, getAuthCode 메서드는 Redis에 인증코드를 조회해서 반환하는 로직입니다.  
만료시간의 경우 저장할 때 300초로 설정해놓았습니다.  

#### ***인증 코드 저장 및 발송 로직***

```java
@Transactional
public void authenticationEmailWithRedis(String email) {

    if(memberRepository.existsByEmail(email)) {
        throw new BaseException(ErrorCode.MEMBER_EXISTS);
    }

    String authCode = this.createCode();

    redisService.setAuthCode(email, authCode);

    asyncService.sendEmail(email, authCode, "[TakeEat] 회원가입 이메일 인증");
}
```
RDB 방식과 달라진 부분은 EmailAuth 엔티티를 저장하는 부분이 없어지고, setAuthCode를 통해 사용자의 email을 key 값으로 그리고 만들어진 인증코드를 value 값으로 하여 바로 Reids에 저장합니다.  

#### ***인증 코드 검증 로직***

```java
    @Transactional(readOnly = true)
    public void validateAuthCodewithRedis(String email, String authCode, BindingResult bindingResult) {
        String findEmailAuthCode = redisService.getAuthCode(email);

        if(findEmailAuthCode == null) {
            bindingResult.rejectValue("authCode", "required", "인증코드 유호시간이 지났거나 발급이 되지 않았습니다.");
        }else if(!findEmailAuthCode.equals(authCode)) {
            bindingResult.rejectValue("authCode", "required", "인증코드를 다시 확인해 주세요.");
        }

    }
```

getAuthCode에 email 값을 통해서 redis에 해당 이메일로 저장되어 있는 AuthCode값을 찾아와서 검증하게 됩니다.  

기존의 RDB 방식은  

```java
EmailAuth findEmailAuth = emailAuthRepository.findTop1ByEmailOrderByCreatedTimeDesc(email).orElse(null);
```
와 같이 Spring Data JPA를 통해 조회하여 검증하는 로직이었습니다.  

</br>

### ***성능 비교***

성능을 비교하기 위해서 저는 Apache Jmeter를 활용하여 많은 건수를 실제로 요청을 보내서 테스트해보기로 했습니다.  

그런데 구글에서 스팸을 방지하기 위해서 일반 계정의 경우에 일일 메일 건수를 500건으로 제한을 두었다고 합니다.  

[gmail 건수 제한](https://support.google.com/mail/answer/22839?hl=ko&sjid=12300065091378962404-AP#zippy=%2C%EB%A9%94%EC%9D%BC-%EC%A0%84%EC%86%A1-%EC%A0%9C%ED%95%9C%EC%97%90-%EB%8F%84%EB%8B%AC%ED%96%88%EC%8A%B5%EB%8B%88%EB%8B%A4)  

그렇기에 테스트를 할 때에는 실제로 이메일 발송하는 로직인 부분은 주석으로 두어서 테스트하기로 했습니다.  
실제로 성능을 비교할 때 해당 부분은 Redis와 RDB의 성능 차이를 비교하는 부분과는 상관이 없는 부분입니다.  

```java
asyncService.sendEmail(email, authCode, "[TakeEat] 회원가입 이메일 인증");
```
해당 부분을 주석처리를 진행하였습니다.  

또한, 인증 코드를 RDB나 Redis에 조회해서 가져오는 부분 또한 따로 분리해서 테스트하기로 했습니다.  

정확히 변경된 부분만을 비교해야 좀 더 정확하게 비교할 수 있을 것 같아 아래와 같이 메서드 2개를 만들었습니다.  

```java
    //RDB 방식으로 인증코드 조회
    @Transactional(readOnly = true)
    public String findAuthCode(String email) {
        log.info("RDB AuthCode Select");
        EmailAuth findEmailAuth = emailAuthRepository.findTop1ByEmailOrderByCreatedTimeDesc(email).orElse(null);
        return findEmailAuth.getAuthCode();
    }

    //Redis 방식으로 인증코드 조회
    @Transactional(readOnly = true)
    public String findAuthCodeWithRedis(String email) {
        log.info("Redis AuthCode Select");
        return redisService.getAuthCode(email);
    }
```

#### ***Controller***

컨트롤러에는 RDB와 Redis에 각각 저장하는 메서드 2개와 또 각각 조회하는 메서드 2개를 만들어 총 4개의 메서드를 만들었습니다.  

#### 인증 코드 저장 메서드

```java
    @PostMapping("/email-send/redis")
    @ResponseBody
    public String sendAuthCode2(@RequestParam("email") String email) {
        emailService.authenticationEmailWithRedis(email);
        return "인증 코드를 발송했습니다.";
    }

    @PostMapping("/email-send")
    @ResponseBody
    public String sendAuthCode(@RequestParam("email") String email) {
        emailService.authenticationEmail(email);
        return "인증 코드를 발송했습니다.";
    }
```

#### 인증 코드 조회 메서드

```java
    @GetMapping("/find/authcode/mysql")
    @ResponseBody
    public String testAuthCode(@RequestParam("email") String email) {
        return emailService.findAuthCode(email);
    }

    @GetMapping("/find/authcode/redis")
    @ResponseBody
    public String testAuthCodeWithRedis(@RequestParam("email") String email) {
        return emailService.findAuthCodeWithRedis(email);
    }
```

#### ***Apache Jmeter 쓰레드 그룹 설정***

Apache Jmeter를 통해서 인증 코드를 저장하는 로직을 RDB와 Redis 각각 2000번씩 호출하여 성능을 비교해보겠습니다.  

그러기 위해서 호출하기 전에 JMeter의 쓰레드 그룹을 설정해줍니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/929b9805-4b57-400e-afb5-2e995d1b9a66" width = 100%></p> 

쓰레드들의 수를 2000개로 할당하고, Ramp-up 시간 즉, 2000개의 요청을 몇초내로 할지 설정하는 시간에 1초로 설정했습니다.  

루프 카운터는 1로 설정하여 반복하지 않도록 해놨습니다.  

#### ***RDB를 통한 저장 로직 호출***

<p align ="center"><img src="https://github.com/user-attachments/assets/8925f8fd-ec84-42d0-9aa7-4a647a425ba3" width = 100%></p> 

그리고 위와 같이 HTTP 요청을 설정합니다.  
컨트롤러에 정의한 RDB 방식의 저장 메서드의 경로를 설정하고, 파라미터 또한 세팅해 줍니다.  

그리고 시작을 누르면 아래와 같이 요약 보고서에 기록이 됩니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/5613fb9f-8e7f-400c-869f-b96c1a5a29b9" width = 100%></p> 

평균 값의 단위는 ms 단위로 2,000건에 대하여 평균적으로 1413ms가 걸리는 것을 확인할 수 있었습니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/aa6892c0-95f8-4041-9f3f-5abc65c94d96" width = 100%></p> 

MySQL에도 실제로 데이터가 저장되는 것을 확인할 수 있었습니다.  

#### Redis 성능 체크

동일하게 2,000건에 대하여 Redis를 사용한 로직을 호출해보겠습니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/789398bd-c2e2-4dd8-ab21-6f6d208a6207" width = 100%></p> 

HTTP요청 URL을 이미지와 같이 변경한 후에 호출해보겠습니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/9a71bdd2-6e71-4852-9251-eaaaf584a4da" width = 100%></p> 

평균적으로 241ms가 걸린 것을 확인할 수 있었습니다.  

저장된 데이터는 아래와 같습니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/a11025ea-f73e-437c-9236-42332b5c728e" width = 100%></p> 

Redis는 동일한 Key값으로는 하나만 저장하기 때문에 가장 마지막으로 저장한 데이터만 저장하게 됩니다.  

그래도 혹시 몰라 아래와 같이 로그를 찍어봤습니다.  

```java
long startTime = System.currentTimeMillis();
redisService.setAuthCode(email, authCode);
long endTime = System.currentTimeMillis();

log.info("Redis 저장 시간 {}ms", endTime - startTime);
```

</br>

<p align ="center"><img src="https://github.com/user-attachments/assets/f4f8f599-b493-4ff3-87f1-1a72e1a34e52" width = 70%></p> 

굉장히 짧은 시간 안에 로그가 찍힌 것을 확인할 수 있었습니다.  

결론적으로 2,000건의 요청에 대해 인증 코드를 저장할 때 RDB는 1413ms가 소요되었고, Redis는 241ms가 소요가 되었으며, 약 5.8배의 차이가 나는 것을 확인했습니다.  

### 인증 코드 조회 비교

다음으로는 동일하게 2,000건에 대하여 조회에 대한 성능을 비교해보겠습니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/d578d12f-9e40-4586-9fae-3791866e2716" width = 70%></p>

조회를 위해 위와 같이 조회를 요청하는 URL로 변경합니다.  
해당 URL은 MySQL 방식으로 이메일 인증 코드를 조회하는 URL입니다.  

#### ***RDB 조회 성능***

<p align ="center"><img src="https://github.com/user-attachments/assets/63bc4d7a-71de-4c19-bd8a-f337091a31a0" width = 100%></p>

위와 같이 2,000건을 RDB 방식으로 인증 코드를 조회하는 API를 요청했을 경우에 평균적으로 314ms가 나오는 것을 확인할 수 있었습니다.  

정확히 말하면 수십번을 테스트하면서 가장 빠르게 나온 경우입니다.  

</br>

<p align ="center"><img src="https://github.com/user-attachments/assets/760d7554-2f19-4943-bce4-58012bfe4310" width = 100%></p>

실제로는 위와 같이 600 ~ 800ms가 많으며, 하드웨어의 성능이 나빠서 그런것인지, 아니면 네트워크의 문제가 있는것인지는 몰라도 1000ms가 넘는 경우도 매우 많이 있었습니다.  
그래도 314ms까지는 성능이 나온다는 것임을 확인했으므로, Redis와 성능 비교할 때에는 314ms를 기준으로 성능을 비교하겠습니다.  

</br>

### ***Redis 인증 코드 조회 성능***

Redis 인증 코드를 조회하기 위해서 HTTP 요청 URL을 다음과 같이 변경합니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/edbf1e11-a6f4-43b5-b927-c3316b150912" width = 100%></p>

</br>

#### ***Redis 조회 성능***

<p align ="center"><img src="https://github.com/user-attachments/assets/416f8492-9bad-4690-9a30-ceb16b0cfcd0" width = 100%></p>

RDB와 동일하게 2,000건에 대하여 성능 테스트를 했을 때 평균 33ms가 나온 것을 확인할 수 있었습니다.  

당연히 Redis 또한 여러번 반복해서 테스트를 해봤습니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/67321cd1-5798-4043-b6fb-6a70de0e0a68" width = 100%></p>

평균속도가 제일 느렸을 때가 75ms이며, 일반적으로 20 ~ 30ms 대의 속도 빈도 수가 많았습니다.  
그래서 어느정도 감안해서 중간값으로는 33ms가 좋을 것 같아 33ms를 기준으로 RDB와 성능을 비교하기로 했습니다.  

아래의 이미지는 Redis 조회 API를 호출했을 때 찍힌 로그들입니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/c6e2d1dc-96d2-4ea3-a114-93826f1bda01" width = 80%></p>

엄청 짧은 시간안에 로그가 찍힌 것을 확인할 수 있었습니다.  

즉, RDB는 314ms가 그리고 Redis의 경우 33ms가 평균적으로 소요가 되었습니다.  
무려 9.5배정도의 속도차이가 나는 것을 확인할 수 있었습니다.  


</br>

### Redis 도입 전과 후 흐름

<table>
	<tbody>
		<tr>
			<th>도입 전</th>
			<th>도입 후</th>
		</tr>
		<tr>
			<td><img width="400px" src="https://github.com/user-attachments/assets/3c8ce4af-adb1-430b-a332-b9c00d02705f" alt="Redis 도입 전"/></td>
			<td><img width="550px" src="https://github.com/user-attachments/assets/fe0ac5da-ce85-4214-88a0-81d8af2742ea" alt="Redis 도입 후"/></td>
		</tr>
    </tbody>
</table>


</br>

## 결론



솔직히 성능 비교를 직접 하기 전에 Redis에 대해 찾아봤을 때 Redis 방식이 속도가 월등하다고들 설명은 했습니다.  

저도 글들을 읽고 Redis가 메모리 기반으로 동작한다고 했을 때 머리로는 이해가 됐지만 눈으로는 직접 보지 않았기에 정말 그정도인가? 라는 의문을 가지고 있었습니다.  

하지만 실제로 프로젝트에 도입하고 성능을 비교해보니 Redis가 월등히 좋다는 것을 확인할 수 있었습니다.  


이전 프로젝트에서 RefreshToken을 저장하기 위해 Redis를 사용한 경험이 있었습니다. 당시에는 성능을 고려하기보다는, Token 값을 임시로 저장하고 쉽게 조회하기 위해 Redis를 선택했었습니다.

하지만 이번에 Redis를 다시 검토하면서, 단순한 선택이 아니라 성능 측면에서도 Redis를 사용하는 이유를 분명하게 이해하게 되었습니다. 머리로만 알고 있던 내용을 직접 확인하면서 Redis의 장점을 더욱 실감할 수 있었습니다.
