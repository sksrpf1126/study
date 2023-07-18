# **_ENUM 대소문자 처리_**

해당 내용은 [사이드 이펙트](https://github.com/Side-Effect-Team/side-effect-backend) 프로젝트와 관련된 내용입니다.

---

## **_배경_**

프로젝트에서 모집 게시글을 검색할 때 기술스택을 포함해서 검색할 수 있도록 구현을 해놨습니다.

```java
    @GetMapping("/scroll")
    public RecruitBoardScrollResponse findScrollRecruitBoard(@Valid @ModelAttribute RecruitBoardScrollRequest request, @LoginUser User user) {
        return recruitBoardService.findRecruitBoards(request, user);
    }
```

와 같이 RecruitBoardScrollRequest타입의 request가 검색 조건의 값들을 받습니다.  
RecruitBoardScrollRequest 클래스는 다음과 같이 정의가 되어 있습니다.

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PRIVATE)
@AllArgsConstructor
public class RecruitBoardScrollRequest {

    private Long lastId;
    @NotNull
    private int size;
    private String keyword;
    private List<StackType> stackTypes;
}
```

여기서 기술스택을 받는 필드는 stackTypes로, StackType이라는 Enum을 List로 받아서 처리하고 있습니다.

StackType은 아래와 같이 정의되어 있는 상태입니다.

```java
public enum StackType {
    JAVASCRIPT, TYPESCRIPT, REACT, VUE, SVELTE, NEXT_JS, NEST_JS, NODE_JS,JAVA, SPRING, GO;
}
```

  <p align = "center">
  <img src="https://github.com/sksrpf1126/study/assets/62879192/a32b5299-311a-4f2d-ab5b-05a58ba7ceda" width = 90%>
  </p>

위와 같이 대문자로 요청을 보낼 떄는 정상적으로 응답하게 됩니다.

  <p align = "center">
  <img src="https://github.com/sksrpf1126/study/assets/62879192/aa7351d3-10b1-4a95-bb9d-ab844696dd07" width = 90%>
  </p>

하지만 위와 같이 소문자로 변경하여 요청을 보내게 되는 경우 400 에러가 반환됩니다.

검색 조건을 실어서 보내는 HTTP 요청에 대해 역직렬화를 거쳐서 객체로 만들어서 필드들의 값을 초기화 할 때 ENUM 타입에 대해서는 대문자나 ENUM타입의 순서를 의미하는 숫자 값만을 인식합니다. 즉, 소문자나 다른 값으로는 인식이 되지 않으며, 400의 Bad Request를 반환하게 됩니다.

그래서 프론트를 맡는 팀원분들한테 API를 요청할 때 ENUM 타입에 대해서는 대문자로 전달해달라고 하였습니다.  
하지만 이후에 소문자로 전달해도 동작되도록 요청을 하셨고, 이를 해결하는 내용을 해당 글에 정리하게 된 것입니다.

---

## **_타입 컨버터_**

### **_타입 컨버터란?_**

```java
@GetMapping("/test")
public String test(@RequestParam Integer data) {
return "ok";
}
```

위와 같은 컨트롤러의 메서드가 존재할 때, localhost:8080/test?data=1 과 같이 요청하게 되면 위의 data의 값에는 Integer 타입으로 1의 값이 들어가게 됩니다.
하지만 실제로는 Query String으로 data=1 부분에서 1은 숫자가 아닌 문자열입니다.

이것이 가능하게 되는 이유는 스프링이 중간에서 타입을 변환해주었기 때문이며, 이를 가능케 한 것이 바로 **_타입 컨버터_** 입니다.

타입 컨버터는 @RequestParam 외에도 @PathVariable 이나 @ModelAttribute 에도 적용됩니다.

그러면 프로젝트에서 사용중인 ENUM 타입에 대해 처리를 해주는 타입 컨버터를 정의해주어서 등록해두면 소문자뿐만 아니라 다른 값이 들어와도 ENUM타입에 매핑시킬 수 있을 것 같습니다.

### **_타입 컨버터 도입_**

**_StackType 코드 추가_**

```java
public enum StackType {
    JAVASCRIPT("javascript"), TYPESCRIPT("typescript"), REACT("react"), VUE("vue"),
    SVELTE("svelte"), NEXT_JS("nextjs"), NEST_JS("nestjs"), NODE_JS("nodejs"),
    JAVA("java"), SPRING("spring"), GO("go");

    private final String value;

    StackType(String value) {
        this.value = value;
    }

    public String getValue() {
        return value;
    }

    public static StackType of(String value) {
        return Stream.of(StackType.values())
                .filter(stackType -> stackType.getValue().equalsIgnoreCase(value))
                .findFirst()
                .orElse(null);
    }

}
```

우선 ENUM의 StackType 코드를 위와 같이 추가했습니다.  
value값을 소문자로 정의해 두었는데, 여기서 소문자 값을 굳이 추가하지 않고 상수값 자체를 equalsIgnoreCase로 비교하면 되지만 이후 소문자가 아닌 한글과 같은 값으로도 가능하게 바꾸기 위해서는 위와 같은 구조가 좋다고 생각했습니다.

코드에서 핵심은 of 메서드입니다.  
해당 전역 메서드는 이후 뒤에서 정의할 타입 컨버터에서 사용할 예정으로, 문자열을 받아서 StackType에 정의해 둔 상수값들중에 일치하는 값이 있으면 해당 값을 반환하고, 하나도 일치하지 않는다면 null값을 반환하도록 하는 메서드 입니다.

**_타입 컨버터 추가_**

```java
@Component
public class StackTypeRequestConverter implements Converter<String, StackType> {

    @Override
    public StackType convert(String source) {
        return StackType.of(source);
    }
}
```

위와 같이 Converter 인터페이스를 구현하고, convert 메서드를 재정의하면 됩니다.  
convert 메서드의 역할은 HTTP 요청으로 넘어온 ENUM(소문자)의 문자열 값을 source라는 변수로 값을 받아서 StackType의 of 메서드에 인자로 전달하여 값을 return 합니다.

#### **_커스텀 타입 컨버터 등록_**

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new StackTypeRequestConverter());
    }
}
```

이후 위와 같이 구현한 타입 컨버터를 추가해주면 이후에 자동으로 스프링에서 ENUM타입에 대해서 타입을 변환할 때 해당 타입 컨버터를 사용하게 됩니다.

#### **_적용 후_**

  <p align = "center">
  <img src="https://github.com/sksrpf1126/study/assets/62879192/bd569f5f-9258-4a8b-8f10-ab19e79eeddd" width = 90%>
  </p>

타입 컨버터를 적용 후에 소문자로 다시 요청을 보낸 결과 400 에러가 아닌 정상적으로 응답이 되는 것을 확인할 수 있었습니다.

## **_@RequestBody 에서는?_**

타입 컨버터는 @PathVariable, @RequestParam, @ModelAttribute 에서만 동작하며, @RequestBody에서는 동작이 되지 않습니다.

```java
    @PostMapping("/scroll")
    public RecruitBoardScrollResponse findScrollRecruitBoard(@Valid @RequestBody RecruitBoardScrollRequest request, @LoginUser User user) {
        return recruitBoardService.findRecruitBoards(request, user);
    }
```

위의 메서드에서 @ModelAttribute를 @RequestBody로 변경하였으며, JSON으로 데이터를 전달해 보겠습니다.

  <p align = "center">
  <img src="https://github.com/sksrpf1126/study/assets/62879192/5b6e0402-7864-4075-b169-f4e6ddb367c7" width = 90%>

대문자로는 정상적으로 호출이 됩니다.

  <p align = "center">
  <img src="https://github.com/sksrpf1126/study/assets/62879192/57506d1d-6b02-45c4-b011-ac71660b243c" width = 90%>

하지만, 소문자로 변경하고 그대로 요청을 해보면 400 에러가 반환되는 것을 확인할 수 있습니다.

그 이유는 역직렬화 하는 방식의 차이에 있는 것으로 알고 있으며, 역직렬화의 하는 방식은 아래의 글에 정리를 해놓았습니다.

[@ModelAttribute와 @RequestBody 역직렬화](<https://github.com/sksrpf1126/study/blob/main/Spring(Spring%20Boot)/%40ModelAttribute%EC%99%80%20%40RequestBody%20%EC%97%AD%EC%A7%81%EB%A0%AC%ED%99%94.md>)

짧게 요약하면, @RequestBody는 보통 JSON의 데이터 형식을 HTTP 본문 데이터로부터 읽습니다.  
자바는 @RequestBody 어노테이션이 있다면, HTTPMessageConveter를 내부적으로 사용하며, 실제로 JSON을 처리하는 클래스는 해당 인터페이스를 구현하는 MappingJackson2HttpMessageConverter 를 사용합니다.

내부적으로 Jackson을 사용하고, Jackson의 ObjectMapper를 통해서 JSON 데이터를 읽고 쓰게 됩니다.  
그리고 읽은 데이터들과 자바의 **_리플렉션 API_** 를 사용하여 역직렬화를 하게 됩니다.

그러면 @RequestBody로 ENUM 타입을 역직렬화 할 때 특정 메서드를 실행시키게 하여 소문자나, 특정 값이 넘어올 경우 값을 비교해서 매칭시키면 될 것이라 생각합니다.

### **_@JsonCreator_**

@JsonCreator는 JSON 데이터에서 객체로 "역직렬화"할 때 사용할 메서드에 쓰이는 어노테이션이라고 보면 됩니다.  
물론, 더 자세히는 생성자나 팩토리 메서드 패턴에서 일반적으로 사용된다고 합니다.

해당 어노테이션은 당연히 ENUM 타입에 대해서 역직렬화 할 때도 사용이 가능합니다.

ENUM의 StackType 클래스에 아래의 메서드를 추가합니다.

```java
    @JsonCreator(mode = JsonCreator.Mode.DELEGATING)
    public static StackType parsing(String value) {
        return Stream.of(StackType.values())
                .filter(stackType -> stackType.getValue().equalsIgnoreCase(value))
                .findFirst()
                .orElseThrow(() -> new InvalidValueException(ErrorCode.STACK_NOT_FOUND));
    }
```

JsonCreator.Mode.DELEGATING는, 인자로 하나만 받을 때 사용합니다.

해당 메서드가 바로 JSON 데이터에서 StackType에 대한 ENUM타입으로 필드값을 초기화 할 때 사용되는 메서드입니다.

#### **_적용 후_**

  <p align = "center">
  <img src="https://github.com/sksrpf1126/study/assets/62879192/ef7a9f26-6398-4a8b-ac13-8e58e13edd16" width = 90%>

@JsonCreator를 적용 후에 소문자로 요청을 다시 보내보면, 400 에러가 아닌 정상적으로 응답이 되는 것을 확인할 수 있습니다.

## **_정리_**

@PathVariable, @RequestParam, @ModelAttribute와 같이 HTTP 파라미터(Query String) 이나 Form 데이터를 처리하는 경우에 역직렬화 하는 과정 중 타입에 대한 처리를 원한다면 **_타입 컨버터_** 를 추가하면 됩니다.

@RequestBody의 경우에는 JSON 데이터가 HTTP 본문으로 넘어오기에 역직렬화를 별도의 컨버터(HTTPMessageConverter) 를 사용하기에 타입 컨버터로는 해결할 수 없고, 역직렬화 하는 과정에 별도의 메서드(로직)을 사용하게 함으로써 해결할 수 있었습니다.
