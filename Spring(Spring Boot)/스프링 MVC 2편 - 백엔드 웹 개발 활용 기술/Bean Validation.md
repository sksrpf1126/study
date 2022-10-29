# **_Bean Validation_**

해당 내용은 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술(김영한님)의 강의를 보고 정리한 내용입니다.

---

Validator 방법보다 더 깔끔한(?) 검증 방법이 바로 **_Bean Validation_** 방법이다.

해당 방법을 사용하기 위해서는

```
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

의존성을 추가해줘야 한다.

</br>

---

## **_사용 방법_**

```java
@Data
public class Item {

    private Long id;

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    @NotNull
    @Max(value = 9999)
    private Integer quantity;

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```

위와 같이 애노테이션을 기반으로 검증을 하는 방법이다.

@NotBlank는 빈값 + 공백인 경우를 허용하지 않는다.

@NotNull 은 null값을 허용하지 않는다.

@Range와 @Max는 값의 범위를 지정한다.

이 외에도 많은 애노테이션이 존재한다.

</br>

```java
    @PostMapping("/add")
    public String addItem(@Valid @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

        //특정 필드가 아닌 복합 룰 검증(가격 * 수량 > 10,000)
        if(item.getPrice() != null && item.getQuantity() != null){
            int resultPrice = item.getPrice() * item.getQuantity();
            if(resultPrice < 10000){
                bindingResult.reject( "totalPriceMin", new Object[]{10000, resultPrice}, null);
            }
        }

        //검증에 실패하면 다시 입력 폼으로
        if(bindingResult.hasErrors()){
            log.info("errors={}",bindingResult);
            //bindingResult는 바로 뷰로 전달되기 때문에 모델에 안담아도 됨
            //pdf 13쪽에 설명
            return "validation/v3/addForm";
        }

        //에러없는 성공로직
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v3/items/{itemId}";
    }
```

기존의 @InitBinder로 WebDataBinder 방식으로 검증기를 추가할 필요가 없다. 검증기를 추가하는 방법에는 main 메서드의 클래스에 글로벌로 추가하여 모든 컨트롤러에 검증기를 사용할 수 있는 방법을 제공해주는데, Bean Validation은 자동으로 글로벌로 검증기를 추가하여 모든 컨트롤러에서 사용할 수 있도록 해주기 때문에 추가할 필요가 없는 것이다.

단, 따로 등록한 글로벌 검증기가 존재한다면 작동이 되지 않으니 유의해야 한다.

Bean Validation에 의한 애노테이션이 동작을 하게 되어 검증 오류가 발생하는 경우에는 FieldError , ObjectError 를 생성해서 BindingResult 에 담아준다.

그렇기에 타임리프 단의 코드는 변경하지 않아도 되는 것이다.

---

**_주의해야 할 점_**

Bean Validation은 바인딩에 성공한 경우에만 동작하게 된다.(애노테이션들)

간단한 예시로

itemName 에 문자 "A" 입력 타입 변환 성공 itemName 필드에 BeanValidation 적용

price 에 문자 "A" 입력 "A"를 숫자 타입 변환 시도 실패 typeMismatch FieldError 추가
price 필드는 BeanValidation 적용 X

와 같이 동작하게 된다.

---

다음으로 위 컨트롤러 메서드를 보면, 복합 룰 검증 로직이 들어가게 된다. 그 이유는 Bean Validation을 보면 필드 검증만 할 뿐 오브젝트 오류에 대해서는 검증할 방법이 없다.

Item Class에 @ScriptAssert(lang = "javascript", script = "\_this.price \* \_this.quantity >= 10000", message = "총합이 10,000원이 넘게 입력하세요") 을 추가하는 방법도 있지만, 해당 기능은 별로 추천하지 않는다.

위처럼 컨트롤러에 로직을 넣거나, 아니면 따로 메서드를 추출하는 방법등으로 해결하는게 좋다.

</br>

---

## **_에러 코드(메시지)_**

BindingResult에 에러코드를 통하여 errors.properties에 등록된 에러 메시지를 활용한 적이 있다.

스프링이 에러 코드를 만들어 주기 때문에 해당 에러코드에 메시지를 대입해서만 적어주면, 가져다가 사용할 수 있는 것이다.

그래서 Level별로 정의하여서 사용을 했었는데, Bean Validation 또한 결국에는 BindingResult에 담기고, 담길 때에 에러 코드를 만든다.

즉, Bean Validation 또한 에러코드를 활용하여 메시지를 사용할 수 있는 것이다.

예시로 NotBlank에서 검증 오류가 발생하면 에러 코드로는

NotBlank.item.itemName //level4  
NotBlank.itemName //level3  
NotBlank.java.lang.String //level2  
NotBlank //level1

4개가 생성된다. 타입에러가 나서 에러메시지를 보여줄 때 만들어준 메시지가

typeMismatch.java.lang.Integer=숫자를 입력해주세요.
typeMismatch=타입 오류입니다.

와 같이 정의한 것 처럼 Bean Validation 또한

NotBlank={0} 공백X  
Range={0}, {2} ~ {1} 허용  
Max={0}, 최대 {1}

와 같이 개발자가 정의해서 메시지를 보여줄 수 있다.

이 외에 @NotBlank(message = "검증 오류 발생 시 보여줄 메시지") 처럼 속성값으로도 메시지를 정의할 수 있는데,

BeanValidation 메시지 찾는 순서는

1. 생성된 메시지 코드 순서대로 messageSource 에서 메시지 찾기
2. 애노테이션의 message 속성 사용 @NotBlank(message = "공백! {0}")
3. 라이브러리가 제공하는 기본 값 사용 (공백일 수 없습니다.)

와 같다.

</br>

---

## **_Bean Validation 한계_**

만약에, 등록 폼에는 위에서 정의한 검증으로 실행하되, 수정 폼에서는 ID 값이 들어오기 때문에 ID에는 NotNull을 추가를 해야하고, 수량 또한 수정시에는 9,999개의 제한 없이 무제한으로 설정할 수 있도록 해야한다면 어떻게 해야할까?

방법은 2가지이다.

1. group 사용
2. Item을 직접 사용하지 않고, ItemSaveForm, ItemUpdateForm 같은 폼 전송을 위한 별도의 모델 객체를 만들어서 사용한다.

1번째 사용방법은 우선 group을 구분하기 위한 용도일 뿐인 인터페이스를 만든다.

```java
public interface SaveCheck {
}

public interface UpdateCheck {
}
```

그리고 Item 클래스에

```java
@Data
public class Item {

    @NotNull(groups = UpdateCheck.class)
    private Long id;

    @NotBlank(groups = {SaveCheck.class, UpdateCheck.class})
    private String itemName;

    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
    @Range(min = 1000, max = 1000000, groups = {SaveCheck.class, UpdateCheck.class})
    private Integer price;

    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
    @Max(value = 9999, groups = SaveCheck.class)
    private Integer quantity;

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```

group으로 속성값을 주어서 구분을 하고 컨트롤러의 파라미터에

```java
등록폼에는 @Validated(SaveCheck.class)
수정폼에는 @Validated(UpdateCheck.class)
```

방식으로 구분을 지어주면 된다. (@Valid는 group 방식 사용이 불가하다.)

하지만 Item 클래스를 보면 매우 복잡함을 느낄 수 있으며, 인터페이스도 각각 만들어줘야 한다.

그렇기에 실무에서는 위 방식을 사용하지 않는다고 한다. 위 이유도 있지만 회원가입과 회원수정을 생각해보면 둘이 들어가는 값부터가 다르다는걸 알 수 있다.

그렇기에 각각의 모델객체를 만들어서 따로 검증을 하는게 좋다는 것이다.
