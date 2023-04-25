# **_Spring Security 인증 인가 방식_**

해당 내용은 프로젝트에서 Spring Security에서의 인증 - 인가에 대해 구현하던 중 발생한 문제를 다룹니다.

---

### **_발생한 문제_**

예를 들어 게시글에 관련된 CRUD를 요청하는 각 API들의 경로들이 동일하면서 HTTP Method를 활용하여 CRUD를 구분하거나, 뒤의 경로만 조금씩 달라지게 됩니다.

이럴 경우 URL에 따른 접근 권한을 설정하여 필터 방식으로 인증 - 인가를 검증하는 방식으로 구현을 하면, URL의 구조를 변경을 하면서 동시에 URL에 대한 접근 권한을 설정하는 부분이 매우 복잡해질 것 같았습니다.

그래서 메서드에 어노테이션을 활용하여 인증 - 인가를 처리하는 방식이 있다는 것을 알게 되었고, 해당 방식을 프로젝트에 도입하기로 했습니다.

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/229751440-acc47be6-6acf-4f09-9ebd-48638a9d66db.png" width = 70%>

위와 같이 @PreAuthorize 어노테이션이나 @Secured 방식으로 API 요청에 대한 권한을 체크하도록 구현했습니다.

그런데 여기서 문제가 발생했습니다.

인증 - 인가에 대해 예외처리를 이전에 공부하면서 알게된 FilterSecurityInterceptor가 메서드 방식 또한 같이 처리를 해주는 줄 알았습니다.  
그래서 아래와 같이 Spring Security의 ExceptionTranslationFilter내부에서 인증과 인가에 대한 예외처리를 커스텀한 클래스를 추가해서 핸들링을 했었습니다.

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/229820940-9e2fabe6-dd2c-4302-9f82-aa94cd3074c5.png" width = 90%>

이후, 메서드 방식 또한 인증과 인가에 대한 예외가 발생한다면 해당 클래스들이 동작이 될 것을 기대했습니다.  
하지만 예상과는 달리 아래의 테스트한 방식대로 동작이 되는것을 확인했습니다.

</br>

### **_테스트_**

테스트로, ROLE_USER 권한만 있는 사용자 계정으로 로그인을 하여 AccessToken을 발급 받은 후 해당 토큰으로 ROLE_ADMIN 권한이 있어야 하는 /test/admin을 호출해 봤을 때 아래의 결과가 나왔습니다.

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/229753965-5a9609db-1bef-4ae5-9679-83f43a28d798.png" width = 90%>

위 방식으로 응답한 부분을 찾아보니 아래와 같이 @ExceptionHandler로 Exception 예외가 발생했을 때 처리하는 로직이 동작이 되었습니다.

```java
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(Exception e) {
        log.error("handleException throw Exception : {}", e);
        return ErrorResponse.toResponseEntity(INTERNAL_SERVER_ERROR, "서버에서 오류가 발생했습니다.");
    }
```

이를 통해서 메서드 방식으로 인증 - 인가를 처리할 때 예외가 발생하면 @RestControllerAdvice가 동작이 되는 것을 알 수 있었습니다.

그래서 저는 왜 이렇게 동작하는지를 알기 위해서 메서드 방식의 경우 어떻게 인증-인가를 체크하는지, 예외가 발생하면 어떻게 되는지를 찾아봤습니다.

</br>

---

## **_Filter 기반과 Method 기반_**

Spring Security에서는 URL방식으로 인가를 처리하는 방식에서는 FilterSecurityInterceptor가 동작하고, 메서드 방식(@PreAuthorize, @Secured 등)으로 인가룰 처리하는 방식에서는 MethodSecurityInterceptor가 동작한다고 합니다.

FilterSecurityInterceptor는 기본적으로는, Spring Security의 필터 체인에서 제일 마지막에 위치해 있습니다. URL에 따른 인가를 처리하며, 예외가 발생했을 경우에는 해당 필터 바로 이전에 존재하는 ExceptionTranslationFilter가 인증 및 인가에 대해 예외처리를 해주게 됩니다.

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/229739671-ad62661a-04cf-4226-a6ec-b964693460c3.png" width = 80%>

위 이미지는 Filter와 Method 기반의 동작 방식을 이미지로 보여줍니다.  
해당 이미지를 통해서 Method 방식의 경우에는 FilterSecurityInterceptor가 아닌 MethodSecurityInterceptor 가 동작되는 것을 확인할 수 있습니다.  
그리고 Filter기반에서는 FilterInvocationSecurityMetadataSource를 통해 URL에 필요한 접근 권한이 무엇인지를 알아내고, Method기반에서는 MethodSecurityMetadataSource를 통해 메서드에 필요한 권한이 무엇인지를 알아냅니다.

이후, 두 방식은 최종적으로 AccessDecisionManager를 통해 판단을 합니다.  
여기서 Method 방식의 핵심은 바로 AOP 기반의 프록시를 활용해서 동작한다는 것입니다.

</br>

---

## **_AOP 기반 Method 방식_**

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/229739632-71b79457-0989-45bc-b933-d22edddee9ff.png" width = 80%>

Method 방식은 위 이미지와 같은 방식으로 동작하게 됩니다.

1. 자동 프록시 생성기(DefaultAdvisorAutoProxyCreator)는 MethodSecurityMetadataSourceAdvisor를 통해 프록시 객체를 만들지 말지를 판단합니다.

```java
 Advisor는 보통 Pointcut과 Advice로 나뉘며, Pointcut은 프록시를 적용해야 할 대상인지 아닌지를 판단하는데 사용하며, Advice는 프록시가 적용될 경우 동작되어야 할 로직이 됩니다.
 위 MethodSecurityMetadataSourceAdvisor 또한 Advisor로 MethodSecurityMetadataSourcePointcut이 Pointcut의 역할을 담당하게 됩니다.
 MethodSecurityInterceptor는 프록시가 적용되었을 경우 동작되어야 할 Adivce들을 가지게 됩니다.
```

2. MethodSecurityMetadataSourceAdvisor는 MethodSecurityMetadataSourcePointcut을 통해서 메서드에 보안이 설정되어 있는지를 탐색합니다. (@PreAuthorize, @Secured 등의 어노테이션)

3. MethodSecurityMetadataSourcePointcut은 탐색된 클래스 및 메서드의 정보를 MethodSecurityMetadataSource 클래스에 정보를 전달합니다.

4. MethodSecurityMetadataSource 클래스는 전달받은 클래스 및 메서드의 정보를 파싱해서 권한에 맞게 처리해야할 Advice를 MethodSecurityInterceptor에 등록합니다.

</br>

### **_프록시 등록 이후 동작방식_**

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/229739657-bf95af0a-0441-4942-b10a-54597c01edaf.png" width = 80%>

프록시가 등록된 이후에 인가처리하는 방식은 위와 같습니다.

1. 요청이 들어오고, 해당 요청을 처리할 메서드에 Advice 등록여부(권한 설정 여부)를 확인하며, 등록되지 않았다면 정상적으로 호출합니다.
2. Advice가 등록이 되어 있다면 이때 MethodSecurityInterceptor를 통해 등록된 Advice 로직을 수행합니다. (인가처리)
3. Advice로직에 의해 인가에 대한 승인이 이루어진다면 정상적으로 호출하며, 아니라면 AccessDeniedException 예외를 발생시킵니다.

</br>

---

## **_문제 해결하기_**

Method 방식의 인가처리가 어떠한 방식으로 동작되고, 예외를 발생시키는지 알았습니다.  
맨 위의 테스트에서 @ExceptionHandler(Exception.class) 가 동작되는 이유는 필터에서 발생한 예외가 아니고, DispatcherServlet 내부에서 발생한 예외이기 때문입니다. (ExceptionHandlerExceptionResolver가 동작되어 버린다)

그러면 메서드 방식의 인가 예외를 핸들링하기 위해서는 @ExceptionHandler(AccessDeniedException.class)를 통해서 예외를 핸들링해주면 될 것이라 판단했습니다.

```java
    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDeniedException(AccessDeniedException e) {
        log.error("handleAccessDeniedException throw Exception : {}", e);

        Authentication auth = SecurityContextHolder.getContext().getAuthentication();

        if(auth == null || auth.getPrincipal().equals("anonymousUser")) {
            return ErrorResponse.toResponseEntity(UNAUTHORIZED, "로그인이 필요한 서비스입니다.");
        } else {
            return ErrorResponse.toResponseEntity(ROLE_NOT_EXISTS, "해당 계정은 권한이 없습니다.");
        }
    }
```

인가 예외가 발생할 경우, 인증된 객체를 가져오며, 인증된 객체가 존재하지 않거나 익명 사용자인 경우에는 401예외를, 객체는 존재하지만 권한이 없는 경우에는 403예외를 반환하도록 했습니다.

</br>

---

## **_테스트_**

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/229827973-31f442d8-26af-443e-86ae-4ff7bc078bcf.png" width = 60%>

위 테스트는 로그인을 하지 않았을 때 권한이 필요한 API를 요청했을 경우의 응답입니다.  
인증을 받을 토큰이 존재하지 않았기 때문에 401예외가 반환되는 것을 확인할 수 있습니다.

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/229827982-f28b8520-0e18-4beb-8f58-12b30dff265a.png" width = 75%>

위 테스트는 ROLE_USER 권한을 가진 사용자 계정으로 ROLE_ADMIN 권한이 필요한 API 요청을 했을 경우의 응답입니다.  
인증을 받은 객체가 존재하기에 401예외가 아닌 403예외를 반환하게 됩니다.

예상한대로 동작되는 것을 확인했으며, 이를 통해 URL 필터 방식과 Method 방식의 차이점을 이해했으며, 프로젝트 내에 필요한 경우에는 두 방식을 적절하게 사용해도 좋을 것 같습니다.

</br>

---

## **_참고_**

https://catsbi.oopy.io/747cd6f5-ae0f-40ac-9cc8-0d4c4c31ddfa#200946bc-1bba-4993-b10d-cb5e8b068c83 -> 메서드 인증-인가 동작방식  
https://wildeveloperetrain.tistory.com/170 -> SecurityInterceptor
