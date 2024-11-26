# ***SSR방식에서의 공통 예외처리***

해당 내용은 [테이크 잇(Take Eat)](https://github.com/pickpong/takeeat) 프로젝트를 진행하며, 정리한 내용입니다.

---

## **_SSR이란?_**

SSR은 Sever Side Rendering의 약어로, ***서버 측에서 페이지를 렌더링하는 방식*** 입니다.  
서버에서 페이지를 렌더링한 후에 클라이언트에게 전달이 되므로, 클라이언트측에서는 추가적인 JavaScript 로딩이 필요하지 않습니다.  

SSR은 초기 로딩 속도가 빠르고, SEO(Search Engine Optimization)에 유리하며, 서버에서 데이터를 처리하므로 보안성이 높다라는 것을 장점으로 얘기할 수 있습니다.  

단점으로는 페이지 이동이 느리며, 페이지에 대한 렌더링 처리를 서버에서 하기 때문에 서버의 부하가 증가하게 됩니다.  

```
CSR이란?
CSR은 Client Side Rendering의 약어로, 클라이언트 측에서 페이지를 렌더링하는 방식을 의미합니다. CSR은 서버로부터 받아온 데이터를 클라이언트에서 동적으로 처리하여 페이지를 렌더링합니다.  

일반적으로는 React, Vue와 같은 기술을 통해 프론트를 구현하고, 서버는 오직 요청에 따라 데이터만 전달하게 됩니다.  
이러면 서버의 부하가 감소하게 됩니다.
```

### SSR방식의 원리
<p align ="center"><img src="https://github.com/user-attachments/assets/eb277fc4-8a39-4fe1-b2dc-b1635598accd" width = 100%></p>  

1. 사용자가 웹 사이트에 요청을 보낸다.  
2. 서버는 요청에 맞게 즉시 렌더링 가능한 HTML파일을 만듭니다.  
3. 클라이언트에 전달하는 순간 이미 렌더링 준비가 되었기 때문에 HTML이 즉시 렌더링이 됩니다.  
4. 브라우저가 JS파일을 다운 받습니다.  
5. 다운로드가 이루어지고 있는 상태에서 사용자는 컨텐츠를 볼 수 있지만 JS 파일이 아직 컴파일이 되어 있지 않아 조작할 수는 없습니다.  
6. 브라우저가 JS FrameWork를 실행합니다.  
7. JS까지 성공적으로 컴파일이 되었다면, 웹페이지는 상호작용이 가능해집니다.  

저희 팀이 진행한 프로젝트에서는 프론트를 전문으로 하는 개발자는 존재하지 않아 ThymeLeaf라는 템플릿 엔진을 활용하여 서버에서 직접 페이지를 렌더링해서 전달하였습니다. 즉, SSR 방식입니다.  

</br>

---

## SSR에서의 공통 예외처리

SSR에서 공통 예외처리를 어떻게 해야하는가? 를 고민한 끝에 제 개인적으로 생각한 방법을 정리하겠습니다.  
당연히 프로젝트에서도 해당 방식으로 공통적으로 예외처리하기로 팀원들과 협의가 완료되었습니다.  

### CSR방식에서는 어떻게 했을까?  
이전 프로젝트에서는 CSR방식으로 프론트 개발자와 백엔드 개발자가 나뉘어져 있었고, 저희 백엔드 개발자는 서버에서 오직 데이터를 요청받고, 데이터를 응답받는 즉, API만을 제공해주면 되었기 때문에 API에 대하여 공통 예외처리만을 진행하면 되었습니다.  

하지만, SSR 방식 그러니까 저희가 진행한 현재 프로젝트에서는 Ajax로 요청에 대한 응답으로 JSON형식으로 데이터를 내려주는 API방식도 존재하였고, 페이지의 요청에 따라 Model에 데이터를 담아서 페이지를 렌더링하는 방식도 존재했습니다.  

하지만 저는 이부분을 간과하고 API방식에 대한 예외처리를 초기에 그대로 사용하였습니다.  

### 초기 예외처리 방식

초기에는 아래와 같이 API의 요청이든, 페이지에 대한 요청이든 구분하지 않고 처리하였습니다.  

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BaseException.class)
    public ResponseEntity<ErrorResponse> handleBaseException(BaseException e) {
        ErrorCode errorCode = e.getErrorCode();
        ErrorResponse errorResponse = ErrorResponse.of(errorCode);

        return new ResponseEntity<>(errorResponse, HttpStatus.valueOf(errorCode.getStatus()));
    }
}
```

@RestControllerAdvice를 통해 해당 클래스를 공통 예외를 처리하는 클래스로 정의하였습니다.  

그리고 @ExceptionHandler(BaseException.class)를 통하여 BaseException의 예외가 발생할 시에 handleBaseException 메서드가 실행되도록 구현하였습니다.  

메서드 내부에서는 ErrorCode라는 것을 사용하며, errorCode 객체를 통해서 errorResponse 객체를 만들어서 ResponseEntity의 ErrorResponse 제네릭타입으로 반환하게 됩니다.  

ErrorCode와 ErrorResponse 그리고 BaseException은 다음과 같습니다.  

#### ***ErrorCode***

```java
public enum ErrorCode {

    MEMBER_NOT_FOUND(400, "U_001", "회원을 찾을 수 없습니다."),
    MEMBER_PASSWORD_MISMATCH(400, "U_002", "비밀번호가 일치하지 않습니다."),
    MEMBER_EXISTS(400, "U_003", "해당 아이디는 사용할 수 없습니다"),
    EMAIL_PROVIDER_MISMATCH_ERROR(400, "U_004", "이메일 또는 제공자가 일치하지 않습니다."),
    MEMBER_UNAUTHORIZED(403, "U_005", "로그인이 필요한 서비스입니다."),
    MEMBER_ROLE_NOT_EXISTS(403, "U_006", "해당 계정은 권한이 존재하지 않습니다"),
    NO_SUCH_AUTH_CODE(500, "U_007", "인증 코드 발급에 문제가 발생했습니다."),

    //... 등등 각종 에러코드 정의

    private final String code;
    private final String message;
    private final int status;

    ErrorCode(int status, String code, String message) {
        this.status = status;
        this.message = message;
        this.code = code;
    }

    public String getMessage() {
        return this.message;
    }

    public String getCode() {
        return code;
    }

    public int getStatus() {
        return status;
    }
}
```

와 같이 ErrorCode는 ENUM으로 구현하였으며, Enum에 들어가는 정보로는 code와 message 그리고 상태코드인 status값이 들어가게 됩니다.  

#### ***ErrorResponse***

```java
@Getter
public class ErrorResponse {

    private String message;

    private String code;

    private ErrorResponse(ErrorCode code) {
        this.message = code.getMessage();
        this.code = code.getCode();
    }

    public static ErrorResponse of(ErrorCode code) {
        return new ErrorResponse(code);
    }

}
```

ErrorResponse는 위와 같이 클래스로 정의하였으며, ErrorCode로부터 message와 code값을 꺼내서 저장하게 됩니다.

#### ***BaseException***

```java
public class BaseException extends RuntimeException {

    private final ErrorCode errorCode;

    public BaseException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
    }

    public ErrorCode getErrorCode() {
        return this.errorCode;
    }

}
```

BaseException의 경우에는 위와 같이 RuntimeException을 상속받는 자식클래스로 정의하였으며, 내부적으로 errorCode를 가지도록 하였습니다.  
또한 BaseException을 상속받는 여러 자식 클래스들을 정의하였으며 대표적인 예시로 AuthExeption, EntityNotFoundException 등을 정의하여 각각의 예외에 맞는 이름으로 클래스명을 정의하였습니다.  

</br>

---

### ***예외처리 테스트***

위와 같이 예외를 처리하도록 구현했을 경우에 어떻게 처리되는지를 확인해보기 위해서 아래와 같이 하나의 메서드를 구현하였습니다.  

```java
    @GetMapping("/test")
    public String test(@RequestParam("errorType") String errorType) {

        if(errorType.equals("api")) {
            throw new BaseException(ErrorCode.MEMBER_EXISTS);
        }
        return "member/loginForm";
    }
```

/member/test 경로로 errorType으로 값이 api로 넘어온다면 BaseException에 ErrorCode.MEMBER_EXISTS값을 담아서 예외를 발생시킵니다.  

#### ***호출***

<p align ="center"><img src="https://github.com/user-attachments/assets/d7c28ec9-ce40-4257-8ca1-3ac800257afa" width = 100%></p>

해당 메서드를 errorType을 api 값으로 전달해서 호출하게 된다면 위와 같이 JSON 형식으로 예외를 처리하여 응답하게 됩니다.  

#### 문제점

CSR방식에서라면 위처럼 JSON 형식으로 에러처리에 대한 값을 준다면, 프론트 개발자가 핸들링하여 각 예외에 적합하게 처리하면 됩니다.  

저희의 프로젝트에서도 AJAX의 호출에 대해서는 위와 같이 JSON 형식으로 응답해주면 error 메서드에서 알맞게 처리만 하면 됩니다.  

하지만 문제는 저희의 프로젝트의 구조는 SSR방식으로, 페이지에 대한 요청에 의해 HTML을 최종적으로 반환해야하는 구조에서 에러가 발생했을 때 위와 같이 JSON으로 응답하게 된다면 사용자에게 그대로 위와 같이 그대로 데이터가 보여지게 됩니다.  

하지만 일반적으로 페이지를 요청하는 경우에 서버나 요청주소와 같은 문제가 생겼을 경우에는 문제에 대해 안내하는 페이지나 대표적인 404에러 페이지와 같이 ***JSON 형식이 아닌 페이지 형식*** 으로 응답하게 됩니다.  

</br>

---

### 해결 방안

그래서 제가 생각한 방안은 페이지를 요청할 때에 처리하는 로직들에서 발생시키는 예외와 AJAX와 같이 API 호출에 대해 처리하는 로직들에서 발생시키는 예외의 방식을 구분해서 처리하기로 했습니다.  

지금까지 위에서 설명한 방식은 AJAX와 같이 API 호출에 대하여 예외를 처리하는 방식이었기 때문에 페이지 요청에 대한 예외를 처리하는 공통 로직이 필요했습니다.  

#### 페이지 요청에 대한 예외 공통 처리

```java
@ControllerAdvice
public class GlobalPageExceptionHandler {

    @ExceptionHandler(ErrorPageException.class)
    public ModelAndView handleErrorPageException(HttpServletResponse response, ErrorPageException e) {
        ErrorCode errorCode = e.getErrorCode();
        response.setStatus(errorCode.getStatus());

        ModelAndView modelAndView = new ModelAndView("/error/" + errorCode.getStatus());
        modelAndView.addObject("message", errorCode.getMessage());

        return modelAndView;
    }

}
```
위와 같이 @ControllerAdvice를 통해 페이지 요청에 대해 처리하는 로직에서 발생시킬 예외에 대하여 처리하는 공통 로직을 구현했습니다.  

발생시킬 예외는 우선 간단하게 ErrorPageException이라는 RuntimeException을 상속받은 클래스로 구현을 했습니다.  

내부 로직을 보면 동일하게 ErrorCode를 사용하는 것을 알 수 있습니다.  

하지만 차이점은 바로 ModelAndView객체를 통해서 ErrorCode에 들어있는 상태코드 값에 맞는 예외페이지를 찾고, 예외 발생에 대한 메시지를 modelAndView에 담아서 최종적으로 반환하게 됩니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/4759bc7e-0001-4334-8832-a65fc8e6d6f7" width = 100%></p>

그리고 위와 같이 상태코드에 맞게 보여줄 에러페이지를 공통으로 만들어두기만 하면, 상태코드에 맞게 각각의 HTML페이지가 렌더링 되어 사용자에게 보여지게 됩니다.  

</br>

---

### ***예외 페이지 테스트***

```java
    @GetMapping("/test")
    public String test(@RequestParam("errorType") String errorType) {

        if(errorType.equals("api")) {
            throw new BaseException(ErrorCode.MEMBER_EXISTS);
        }else if(errorType.equals("page")) {
            throw new ErrorPageException(ErrorCode.ORDER_UNAUTHORIZED);
        }

        return "member/loginForm";
    }
```

위에서는 api값일 때에만 BaseException을 발생시켰지만, 이번에는 page값일 때에 ErrorPageException을 발생시키도록 로직을 추가하였습니다.  

ErrorPageException에 ErrorCode.ORDER_UNAUTHORIZED는 Http Status값이 403입니다.  

그리고 403.html은 다음과 같이 작성되어 있습니다.  

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>TakeEat</title>
    <link href="/images/fav.png" rel="shortcut icon" type="image/x-icon">
</head>
<body>
    <h1>403 - Forbidden</h1>
    <p th:text="${message}"></p>
</body>
</html>
```
해당 예외가 발생된다면 위의 403.html이 정상적으로 사용자에게 렌더링이 되어 보여지게 될 것입니다.  

#### 결과

http://localhost:8090/member/test?errorType=page 경로로 
errorType값을 page로 전달하도록 하였습니다.  

이후 응답이 되어 보여지는 화면은 아래와 같습니다.  

<p align ="center"><img src="https://github.com/user-attachments/assets/9857cca0-f8d2-429a-8ae6-5aaa495df2c9" width = 100%></p>

정상적으로 403.html이 사용자에게 전달된 것을 확인할 수 있었습니다.  

## ***정리***

하면서도 위 방식이 정말 맞는 것인지, 아니면 더 좋은 방식이 있는 것인지는 많이 찾아봤지만 명쾌한 답이 없어서 저가 생각한 방식으로 해보자! 하고 구현하였습니다.  

위처럼 API에 대한 요청을 처리하는 로직에서 발생시킬 예외는 BaseException으로  
페이지 요청에 대해 처리하는 로직에서는 ErrorPageException을 발생시키도록 하여 각각 다르게 예외를 처리하여 보여지도록 해놓았습니다.  

최종적으로 팀원들에게도 공유를 하여 위 방식으로 진행하기로 결정을 내렸습니다.  
이를 통해 좀 더 유연하게 개발자가 용도에 맞게 예외를 보여지도록 가져갈 수 있었습니다.  

하지만, 예외를 공통으로 처리하는 클래스가 2개로 나뉘어져 있으며, 좀 더 프로젝트가 복잡해진다면 개발자들간에도 혼란을 줄 수 있다고 생각이 듭니다.
