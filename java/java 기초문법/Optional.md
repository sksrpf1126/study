# **_Optional_**

## **_옵셔널이란?_**

옵셔널은 java8에서 등장한 문법으로, 옵셔널 등장하기 이전에는 NPE(NullPointerException) 예외발생을 막기위해서 if ~ else문으로 null체크를 했었다. 또한, if~else문으로 null체크하는 로직을 보고 이해해야 null을 반환할 수 있는 부분이구나 라고 판단이 된다.

하지만 Optional의 등장으로, if~else문으로 null체크하는 부분을 간단한 로직으로 해결할 수 있으며, 또한 반환타입을 옵셔널로 한다면, 명확히 해당 메서드가 null을 반환할 수 있다는 것을 바로 판단할 수 있다.

실제로 옵셔널은 **_"null을 반환하면 오류가 발생할 가능성이 매우 높은 경우에 '결과 없음'을 명확하게 드러내기 위해 메소드의 반환 타입으로 사용되도록 매우 제한적인 경우로 설계"_** 되었다는 것이다.

</br>

---

## **_옵셔널 관련 메서드_**

### **_of()와 ofNullable()_**

객체를 옵셔널로 래핑할 때 사용하는 메서드는 위 2개가 존재한다.  
하지만 둘의 차이점이 존재하는데, of()의 경우 감사는 객체가 null값을 가지고 있다면 NPE를 발생시켜버린다.  
ofNullable()는 대상이 되는 객체가 null이든 아니든 상관없이 옵셔널로 래핑한다.

```java
@Test
@DisplayName("of() vs ofNullable()")
public void optionalObjCreate() {
    User user = null;
//        Optional<User> optionalUser = Optional.of(user); //of()는 매개변수가 null을 가지고 있다면 NPE 발생
//        Optional<User> optionalUser2 = Optional.ofNullable(user);
}
```

of메서드는 대상 객체가 null일 때 NPE를 강제로 발생시켜 핸들링하려는 의도가 있을 때 사용하면 좋을 것 같고, 평소에는 ofNullable 메서드를 사용하자.

</br>

### **_optional 초기화_**

optional을 초기화할 때는 어떻게 해야하는가?

1. null값으로 초기화
2. Optional.empty()로 초기화

```java
    @Test
    public void 옵셔널초기화() {
        Optional opt1 = null;
        Optional opt2 = Optional.empty();

//        opt1.orElse("default"); //NPE 발생
        opt2.orElse("default"); //정상 작동
    }
```

옵셔널의 의도자체가 null로 반환을 해버리는것을 해결하고자 등장한것이다, 그런데 옵셔널에도 null을 담아버리면 옵셔널의 의미가 없어져 버리기 때문에 초기화를 해야 한다면 empty()를 사용하자.

</br>

### **_다양한 메서드들.._**

위는 매우 기본적인 메서드들만 본 것이며, 옵셔널은 많은 메서드들을 제공한다.

- Optional.get()  
  -> 옵셔널이 래핑한 객체를 꺼내는 메서드로, 해당 객체가 null일 때 get()를 호출하면 NoSuchElementException 예외가 발생해버린다. 그렇기에 orElse(), orElseGet(), orElseThrow()를 적극 사용하자.

- orElse(), orElseGet(), orElseThrow()  
  -> orElse는 null일 경우에 매개변수로 지정해준 값을 대신 반환해주는 메서드이며, orElseGet 메서드는 람다식으로 값을 지정할 수 있는 메서드, orElseThrow 메서드는 null일 때 발생시킬 예외를 람다식으로 지정할 수 있는 메서드이다.

- isPresent()  
  -> 옵셔널이 래핑한 객체가 null일 경우 false, 아닐 경우 true를 반환하는 메서드

이 외에도 여러 메서드들이 존재한다.

</br>

---

## **_옵셔널을 잘 사용하려면_**

- Optional 변수에 Null을 할당하지 말아라
- 값이 없을 때 Optional.orElseX()로 기본 값을 반환하라
- 단순히 값을 얻으려는 목적으로만 Optional을 사용하지 마라
- 생성자, 수정자, 메소드 파라미터 등으로 Optional을 넘기지 마라
- Collection의 경우 Optional이 아닌 빈 Collection을 사용하라
- 반환 타입으로만 사용하라

위 내용은 맨 아래의 참고글들에서 가져온 내용이다.

</br>

### **_Optional 변수에 Null을 할당하지 말아라_**

이 부분은 방금 위에서 설명한 내용으로, 옵셔널 자체에 null을 저장해서 바로 반환해버리면, 해당 옵셔널에 대해서도 Null의 여부를 검사해야 하는 문제가 발생해버리기 때문이다.

</br>

### **_값이 없을 때 Optional.orElseX()로 기본 값을 반환하라_**

get을 사용하면 null일 경우 NoSuchElementException 예외가 발생해버리기 때문에 isPresent()로 한번 체크 후에 get으로 꺼내는 경우가 존재한다. 결국 if ~ else문과 다를게 없기 때문에 해당 방법 보다는 orElseX 로시작하는 메서드들을 적극 활용하자.

</br>

### **_단순히 값을 얻으려는 목적으로만 Optional을 사용하지 마라_**

```java
// AVOID
public String findUserName(long id) {
    String name = ... ;

    return Optional.ofNullable(name).orElse("Default");
}

// PREFER
public String findUserName(long id) {
    String name = ... ;

    return name == null
      ? "Default"
      : name;
}
```

해당 코드는 아래 참고글의 망나니 개발자님 블로그의 코드를 가져온 것이다.

Avoid부분의 메서드를 보자. Optional 메서드를 활용해서 null이 아니면 name을 반환하고, null이면 기본 값인 Default를 반환한다.  
그럼 해당 메서드를 호출하는 쪽은 그냥 String타입으로 받게 될 것이다.  
어떻게 보면 NPE를 발생시키지 않는 좋은 코드처럼 보인다. 하지만 Optional은 Optional 자체를 반환타입으로 사용하여, 해당 메서드에서 Null이 반환될 수 있음(결과 없음)을 판단할 근거가 되며, 호출한 쪽에서 이를 컨트롤을 하는게 적합하다.  
즉, 옵셔널 설계를 의도하지 않은 방향으로 사용한 것이다.

```
꼭 설계의도와 맞게 사용해야하는가?
옵셔널은 객체를 감싸는 객체이다. 즉, 옵셔널을 사용하는 순간 추가적으로 자원이 소모가 된다. 이를 룰을 정하지 않은 채 마구잡이로 사용하다보면 자원이 크게 낭비가 된다는 것이다. 그러므로, 명확히 룰을 정해서 사용해야 하며 그 룰은 설계의도라고 봐도 무방하다.
```

</br>

### **_나머지 3개_**

- 생성자, 수정자, 메소드 파라미터 등으로 Optional을 넘기지 마라
- Collection의 경우 Optional이 아닌 빈 Collection을 사용하라
- 반환 타입으로만 사용하라

왜 한번에 정리를 하냐면, 결국 위 3개의 결론은 동일하다.  
바로 **_"옵셔널은 반환타입으로 써라"_** 이다.

생성자, 수정자, 메서드의 파라미터로 옵셔널을 받는 행위는 이를 받도록 하는 메서드를 개발하고 사용하는 입장에서 해당 옵셔널 자체가 null일 경우도 염두해야 하기 때문에 옵셔널 의미가 없어진다.

다음으로, 컬렉션 클래스들은 자체적으로 비어 있는지 아닌지를 판별할 수 있는 메서드가 있기 때문에 굳이 컬렉션 객체들을 옵셔널로 또 감쌀 필요가 없다는 것이다. (자원 낭비)

마지막으로, 반환 타입으로만 사용하라는 지금까지 정리한 내용들에 의해서 도출된 결론이다.

</br>

---

## **_정리_**

옵셔널은 명확히 어떨 때 사용할지 정해서 사용해야한다. 그것은 보통 반환타입으로만 사용할 때로 정한다. 그 이유는 설계자체가 그렇게 사용하기 위해서 등장한 것이다.

만약 어떤 프로젝트에서 옵셔널을 반환타입 뿐만 아니라 원하는 곳에서 마구잡이로 사용한다고 가정하면, 자원 낭비는 기본이며 다른 협업하는 사람들에게도 혼란을 가져오게 된다.

회사마다 코딩 컨벤션을 왜 정하고 지키겠는가?  
혼자개발하고, 이후에도 혼자 유지보수를 한다면 문제가 없겠지만, 대부분의 프로젝트는 그렇지가 않다. 각자 자신만의 코딩 스타일로 작성해버린다면 이후 유지보수에서부터 한숨이 나올 것이다.

옵셔널도 마찬가지라고 생각하면 된다.
</br>

---

# **_참고_**

https://mangkyu.tistory.com/203 ->망나니 개발자 블로그  
https://www.latera.kr/blog/2019-07-02-effective-optional/#%EA%B0%9C%EC%9A%94
