# **_Validator_**

해당 내용은 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술(김영한님)의 강의를 보고 정리한 내용입니다.

---

서버는 클라이언트가 보낸 데이터를 항상 의심해야 한다.

그렇기에 검증은 필수로 이루어져야 하며, 보통 클라이언트(http request)에서 넘어온 데이터는 컨트롤러단에서 검증을 처리하고, 내부(DB, 서버 -> 서버 호출)에서는 보통 서비스단에서 검증을 한다. (상황에 맞게)

해당 내용은 클라이언트에서 보낸 데이터를 검증하는 방법을 설명한다.

</br>

---

## **_BindingResult 사용_**

BindingResult는 인터페이스로 Error 인터페이스를 상속받는다.

그렇기에 Error으로 변경하여도 동일하게 동작한다는 것을 알아두자.

```java
//    @PostMapping("/add")
    public String addItemV4(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

        //상품명에 글자가 없으면
        if(!StringUtils.hasText(item.getItemName())){
//            bindingResult.addError(new FieldError("item", "itemName", item.getItemName(), false, new String[]{"required.item.itemName"}, null, null));

            //위 부분을 아래로 간단히 가능. rejectValue 내부에 new FindError가 동작
            //그리고 bindingResult는 이미 자신이 타겟을 할 객체를 알고 있으므로 field만 넘기면 됨
            bindingResult.rejectValue("itemName", "required");
        }
        if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
//            bindingResult.addError(new FieldError("item", "price", item.getPrice(),false, new String[]{"range.item.price"}, new Object[]{1000,1000000}, null));
            bindingResult.rejectValue("price", "range", new Object[]{1000,1000000}, null);
        }

        if(item.getQuantity() == null || item.getQuantity() >= 9999){
//            bindingResult.addError(new FieldError("item", "quantity", item.getQuantity(), false, new String[]{"max.item.quantity"}, new Object[]{9999}, null));
            bindingResult.rejectValue("quantity", "max", new Object[]{9999}, null);
        }

        //특정 필드가 아닌 복합 룰 검증(가격 * 수량 > 10,000)
        if(item.getPrice() != null && item.getQuantity() != null){
            int resultPrice = item.getPrice() * item.getQuantity();
            if(resultPrice < 10000){
//                bindingResult.addError(new ObjectError("item", new String[]{"totalPriceMin"}, new Object[]{10000, resultPrice}, null));
                bindingResult.reject( "totalPriceMin", new Object[]{10000, resultPrice}, null);
            }
        }

        //검증에 실패하면 다시 입력 폼으로
        if(bindingResult.hasErrors()){
            log.info("errors={}",bindingResult);
            //bindingResult는 바로 뷰로 전달되기 때문에 모델에 안담아도 됨
            //pdf 13쪽에 설명
            return "validation/v2/addForm";
        }

        //에러없는 성공로직
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v2/items/{itemId}";
    }
```

우선 컨트롤러의 파라미터에서는 @ModelAttribute Item item, BindingResult bindingResult 와 같이 검증할 대상 이후에 BindingResult를 선언해주어야 한다.

BindingResult가 없을 때에 Item 객체에 타입에러가 나는 경우에는 400번대(405?)에러를 반환하여서 클라이언트에게 에러화면을 뿌려준다. 즉, 컨트롤러가 정상적으로 호출이 되지 않는다.

하지만 위처럼 BindingResult를 선언해두면, Item 객체 내부에서 발생한 에러를 클라이언트에게 반환하지 않고, BindingResult에 저장을 시키고, 이후 정상적으로 컨트롤러를 호출시킨다.

즉, BindingResult는 자신이 타겟으로 할 객체를 알고 있기 때문에 가능한 것인데, 이는 Target객체가 바로 앞에서 선언이 되었기 때문이다.

그래서 파라미터 순서가 중요하다.

bindingResult.rejectValue()로 개발자가 직접 에러를 넣어줄 수 있다. 그리고 이를 타임리프에서 간단하게 꺼내서 쓸 수 있기 때문에 화면단도 쉽게 처리가 가능하다.

하지만 컨트롤러 메서드 하나에 검증로직과 성공로직이 같이 있다보니 메서드의 책임이 커지며 유지보수에도 어려움이 느껴진다.

해당 로직들을 분리해보자.

</br>

---

## **_Validator 인터페이스 활용_**

스프링에서 제공하는 Validator 인터페이스를 활용해보자.

```java
@Component
public class ItemValidator implements Validator {
    @Override
    public boolean supports(Class<?> clazz) {
        //파라미터로 넘어온 clazz가 Item클래스를 지원하냐
        //Item == clazz or Item == 자식클래스
        return Item.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        Item item = (Item) target;
        //상품명에 글자가 없으면
        if(!StringUtils.hasText(item.getItemName())){
            errors.rejectValue("itemName", "required");
        }
        if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            errors.rejectValue("price", "range", new Object[]{1000,1000000}, null);
        }

        if(item.getQuantity() == null || item.getQuantity() >= 9999){
            errors.rejectValue("quantity", "max", new Object[]{9999}, null);
        }

        //특정 필드가 아닌 복합 룰 검증(가격 * 수량 > 10,000)
        if(item.getPrice() != null && item.getQuantity() != null){
            int resultPrice = item.getPrice() * item.getQuantity();
            if(resultPrice < 10000){
                errors.reject( "totalPriceMin", new Object[]{10000, resultPrice}, null);
            }
        }
    }
}
```

컨트롤러에서 사용하기위해서 해당 클래스에 @Component를 사용하였다.

supports()는 검증 대상을 구분하기 위한 용도로 사용한다. (뒤에서 자세히 설명)

그리고 validate()에서는 Object타입으로 target을 받으며, Error타입으로 BindingResult를 받을 것이다.

이후 형변환을 하여, 기존에 작성한 검증 로직을 그대로 옮겨왔다.

```java

    //DI
    private final ItemValidator itemValidator;

    /**
     * 컨트롤러가 호출이 될 때마다 해당 @InitBinder가 실행이 되어 itemValidator 검증기를 WebDataBinder에 넣어둔다.
     * 사용자의 요청이 올 때마다 WebDataBinder를 만들어서 검증기를 추가함.
     */
    @InitBinder
    public void init(WebDataBinder dataBinder){
        dataBinder.addValidators(itemValidator) ;
    }

    //@Validated에 의해서 넣어둔 검증기가 실행이 됨. (@Validated 대신 @Valid 사용가능 대신 의존성 추가해줘야 함)
    @PostMapping("/add")
    public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

        //검증에 실패하면 다시 입력 폼으로
        if(bindingResult.hasErrors()){
            log.info("errors={}",bindingResult);
            //bindingResult는 바로 뷰로 전달되기 때문에 모델에 안담아도 됨
            //pdf 13쪽에 설명
            return "validation/v2/addForm";
        }

        //에러없는 성공로직
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v2/items/{itemId}";
    }
```

@InitBinder에 의해서 해당 컨트롤러가 호출이 될 때마다 WebDataBinder를 만들어서 검증기를 추가한다.

그리고 추가한 검증기를 사용하고자 하는 메서드는 @Validated 또는 @Valid 애노테이션만 걸어주면 끝이다.

하지만 검증기가 여러개가 추가가 되었다면, 어떠한 검증기를 사용할지에 문제가 생긴다.

하지만 고민할 필요가 없다. 예를 들어 검증기에 User객체를 검증하는 검증기를 추가하여, 현재 컨트롤러에 추가하 검증기가 2개가 있다고 가정하자.

@Validated에 의해 검증기를 찾아서 수행하는데, 찾을 때에 바로 supports()가 동작이 된다.

클래스 타입을 비교하여 실행할 수 있는 검증기를 찾아서 수행하게 되는 것이다.

위 방식도 좋아보이지만, 더 효율적이고 깔끔한 방식이 존재한다.

바로 Bean Validation 기술을 활용하는 것이다.  
 해당 기술은 따로 간단히 정리할 예정이다.
