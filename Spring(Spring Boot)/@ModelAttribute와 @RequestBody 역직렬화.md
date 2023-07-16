# **_@ModelAttribute와 @RequestBody 역직렬화_**

해당 글은 @ModelAttribute와 @RequestBody의 사용법에 익숙한 경우를 가정하여 데이터의 **_역직렬화_** 에 초점을 맞춰서 정리한 내용입니다.

---

## **_용어 정리_**

**_@ModelAttribute_** 어노테이션은 기본적으로 HTTP 파라미터 즉, Query String의 데이터나 Form 형태의 데이터를 객체형태로 받기 위해서 사용하는 어노테이션입니다.

다음으로, **_역직렬화_** 는 HTTP를 통해 요청에서 전달해주는 HTTP 파라미터(Query String), HTTP 요청 본문 데이터 (HTTP body로 전달되는 JSON 또는 XML 등의 형태)를 JAVA 오브젝트(객체)로 변환하는 것을 의미합니다.

**_직렬화_** 는 역직렬화의 반대의 의미로, JAVA 오브젝트를 JSON과 같은 HTTP에 실어서 보낼 수 있는 데이터로 변환하는 것을 의미합니다.

---

## **_@ModelAttribute 역직렬화 흐름_**

일반적으로 @ModelAttribute가 역직렬화를 하는 방식은 아래와 같이 정해져 있다고 합니다.

@ModelAttribute 를 달아준 객체에 요청 파라미터의 데이터를 담을 때는 아래의 순서로 진행됩니다.

- 적절한 생성자를 찾아서 새 인스턴스를 생성한다.
  - public 으로 선언된 생성자를 찾는다.
    - 찾은 생성자가 없다면, public 이 아닌 생성자 중에 매개변수 개수가 제일 적은 생성자를 선택한다. (보통 기본 생성자)
    - 찾은 생성자가 1개라면, 해당 생성자를 선택한다.
    - 찾은 생성자가 여러 개라면, 매개변수 개수가 제일 적은 생성자를 선택한다.
  - 선택한 생성자를 이용하여 인스턴스를 만들 때 생성자의 매개변수 이름 중 클라이언트가 요청한 파라미터 이름과 같은 것이 있다면 해당 매개변수에다가 요청한 파라미터의 값을 넣어 생성한다.
  - 클라이언트가 요청한 파라미터들을 기준으로 setter 메서드를 찾아서 실행시킨다. (생성자 생성을 통해 값을 넣은 것과 상관없이)

해당 내용은 https://blog.karsei.pe.kr/59 블로그를 참고하였으며, 해당 글에서는 저렇게 동작하는 이유를 내부적인 소스를 분석 및 해석까지 해놓았습니다.

### **_코드로 알아보자_**

```java
public class ModelAttributeDto {
    private String name;
    private int age;
}
```

간단하게 위와 같은 구조를 가지는 클래스가 존재합니다.

```java
@GetMapping("/model")
public ModelAttributeDto modelAttributeTest(@ModelAttribute ModelAttributeDto dto) {
    log.info("modelAttributeDto = {}", dto);
    return dto; //getter가 없기에 현재는 return이 제대로 안됨
}
```

그리고 위와 같이 @ModelAttribute 어노테이션을 통해 ModelAttributeDto 객체를 받도록 해놓았습니다.

그리고 만약에 localhost:8080/model?name=테스트&age=10 과 같이 Query String 형태로 요청하면 어떻게 될까요?

  <p align = "center">
  <img src="https://github.com/sksrpf1126/spring-annotation/assets/62879192/be0a781d-282f-4614-9c1a-39d9ce06b391" width = 90%>
  </p>

위와 같이 PostMan으로 요청을 합니다.

  <br>

  <p align = "center">
  <img src="https://github.com/sksrpf1126/spring-annotation/assets/62879192/8f0a53de-9518-4e07-aba1-1114bace2bf8" width = 70%>
  </p>
  
위는 디버깅을 한 상태로, 객체는 생성되었지만 객체의 필드의 name값과 age값이 들어가지 않은 것을 확인할 수 있습니다.

기본 생성자를 토대로 객체까지는 생성하였지만, 필드들의 값은 null이거나 기본값(0) 으로 초기화만 하고 요청할 때 전달한 값들은 들어가지 않았습니다.  
그 이유는 Setter 메서드들이 선언되어 있지 않았기 때문입니다.

### **_Setter 추가_**

```java
@Setter
public class ModelAttributeDto {
    private String name;
    private int age;
}
```

위와 같이 Setter를 추가하고 똑같이 요청을 하면 이제 데이터가 정상적으로 들어가지게 됩니다.  
하지만 DTO는 일반적으로 불변 객체형태로 만드는 경우가 있습니다. 즉, 데이터의 값을 읽을 수는 있지만 변경하지는 못하도록 setter 메서드를 제한합니다.

그러면, 위의 상황에서 setter 메서드 없이 어떻게 해야할까요?

### **_생성자 활용_**

위에서 생성자가 하나라면 해당 생성자로 객체를 생성한다고 하였습니다.  
그러면, 전체 필드를 초기화하는 생성자 하나를 선언하면 해결될 것 같습니다.

```java
@AllArgsConstructor
public class ModelAttributeDto {
    private String name;
    private int age;
}
```

@AllArgsConstructor는 전체 필드를 받아서 객체를 생성하는 생성자를 추가해주는 어노테이션으로 위를 추가하고 요청하면 정상적으로 필드의 값이 들어가게 됩니다.

즉, @ModelAttribute는 Query String이나 Form에서 요청한 데이터를 생성자와 setter를 사용하여 역직렬화 즉, 객체로 만드는 것을 확인할 수 있었습니다.

---

## **_@RequestBody 역직렬화_**

@RequestBody의 경우 JSON이나 XML 형태의 데이터를 역직렬화를 통해 자바 객체를 생성하게 됩니다.  
즉, @ModelAttribute의 Query String이나 Form 형식과는 전혀 다른 형태의 데이터들을 받는다는 것이고, 다른 방식으로 역직렬화를 한다는 것입니다.

해당 내용은 일반적으로 많이 쓰이는 JSON 형태의 데이터를 예시로 설명합니다.

### **_Jackson_**

@RequestBody 어노테이션이 있다면 스프링은 HTTPMessageConverter를 내부적으로 사용합니다. 해당 클래스는 인터페이스이며, 실제로 JSON을 처리하는 클래스는 해당 인터페이스를 구현하는 MappingJackson2HttpMessageConverter 를 사용합니다.  
이름을 보면 알 수 있듯이, 해당 구현체는 내부적으로 Jackson을 사용하며 Jackson의 ObjectMapper를 통해서 JSON데이터를 읽고 씁니다.

하지만 해당 방식은 그저 JSON이라는 데이터를 읽고 쓸 수 있도록 처리할 뿐이지 실제로 자바 객체로 변환하는 "역직렬화" 를 할 수는 없습니다.  
그래서 역직렬화를 하기 위해 사용하는 기술이 **_리플렉션 API_** 입니다.

### **_리플렉션 API_**

**_리플렉션 API_** 란, 구체적인 클래스의 타입을 알지 못해도 해당 클래스의 메타 데이터(메서드, 타입, 변수 등)만으로 해당 클래스를 활용할 수 있도록 하는 API 입니다.

활용의 예로는 일반적으로는 자바에서 객체를 생성하기 위해서는 new 연산자를 사용하지만, 리플렉션은 그저 클래스의 필드들의 정보, 정의된 메서드들의 정보만으로 객체를 생성할 수 있습니다. 그렇기에 private으로 생성자를 막아둔다 하여도 리플렉션 기술로는 객체를 생성할 수 있습니다.

핵심은 클래스의 정보만으로 객체를 생성한다는 것입니다.

```
리플렉션 API라고 클래스의 모든 정보를 사용하지는 못합니다.
객체의 필드나 메서드의 정보를 토대로 마음대로 객체를 조작할 수는 있지만 "생성자의 인자 정보" 만은 리플렉션 또한 마음대로 할 수가 없습니다.
그래서 리플렉션은 기본 생성자를 토대로 객체를 생성하고 이후 클래스의 정보들을 토대로 필드들의 값을 넣거나 메서드를 호출합니다.
더 자세히 설명하면 Java 8 이전에는 리플렉션 기술로 메서드의 인자정보를 가져올 수 없었다고 합니다.
Java 8 이후 부터는 파라미터의 정보를 가져올 수 있게 되어 생성자의 인자 정보 또한 가져올 수 있게 되었지만 따로 설정을 해주어야 하며, 이 의미는 기본적으로는 기본 생성자를 지금도 사용하고 있다는 의미입니다.

더 자세한 내용은 https://colour-my-memories-blue.tistory.com/16 글 참고
```

### **_@RequestBody 역직렬화 흐름_**

@RequestBody는 앞서 말했듯이 Jackson과 리플렉션 API를 통해 역직렬화를 하게 됩니다.  
리플렉션 기술에서는 getter 또는 setter 둘 중에 하나가 필수입니다. setter가 아닌 getter로도 가능한 이유는 스프링에서 리플렉션을 사용하는 원리에 있습니다.

일반적으로 getter와 setter는 getName, setName과 같이 get이후에 필드(프로퍼티) 명, set이후에 필드명이 오게됩니다.  
스프링 내부에서는 위에서 get이나 set의 이름을 지우고 필드명만을 가져와서 JSON데이터의 Key값과 매칭이 된다면 해당 Key값의 Value를 객체의 필드에 넣게 됩니다.

흐름은 아래와 같습니다.

1. 기본 생성자로 객체를 생성한다.
2. Jackson을 통해 읽은 JSON 데이터는 Key - Value 형태이며, Key값을 활용하여 클래스의 Getter 또는 Setter의 앞의 getXXX, setXXX에서 get과 set을 제외하고 XXX의 프로퍼티(필드)명과 JSON데이터의 Key값을 비교하여 일치한다면 해당 프로퍼티에 값을 넣는다.
3. 생성된 객체를 컨트롤러의 해당 객체타입의 참조변수와 연결한다.

### **_코드로 알아보자_**

```java
public class RequestBodyDto {
    private String name;
    private int age;
}
```

의 클래스가 존재하며 아래는 컨트롤러입니다.

```java
@PostMapping("/request")
public String requestBodyTest(@RequestBody RequestBodyDto dto) {
    log.info("requestBodyDto = {}", dto);
    return "success";
}
```

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/6ab5f514-1567-44e0-a046-558e20b2703e" width = 90%>
</p>

위는 JSON 형식으로 요청하는 경우입니다.

  <p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/cb95224e-299e-4869-9f0b-04cba51e9515" width = 90%>
</p>

다음으로 위는 요청에 대한 디버깅을 한 경우입니다.

생성자는 존재하지만, Getter와 Setter 둘 다 존재하지 않기 때문에 필드 값은 정상적으로 초기화 되지 않았습니다.

### **_Getter + private 기본 생성자_**

```java
@Getter
public class RequestBodyDto {
    private String name;
    private int age;

    private RequestBodyDto(){}

}
```

위와 같이 Getter를 추가하고, 생성자의 접근 제어자를 private으로 했습니다.  
간혹, 객체 생성을 막기 위해서 기본 생성자의 접근 제어자를 private으로 해두는 경우가 있습니다.  
그러면 다시 동일한 요청을 해보겠습니다.

  <p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/c27d283c-ce95-477b-8ea0-db16efdaa59f" width = 90%>
</p>

위는 요청에 대한 디버깅으로 정상적으로 객체가 생성되고, 필드의 값들도 정상적으로 초기화가 된 것을 확인할 수 있습니다.

이를 통해 Getter로도 필드의 값을 초기화 할 수 있음을 알 수 있으며, 리플렉션 기술은 생성자의 접근 제어자에 영향이 없다는 것을 확인할 수 있었습니다.

### **_알아두면 좋은 내용_**

위에서 리플렉션의 내용을 설명하면서 JAVA 8 부터는 생성자의 인자 정보 또한 활용할 수 있다고 했습니다.  
단, 설정이 필요하다고 했습니다.  
알아보니 무조건 필요한 것은 아니고 Intellij는 설정이 필요하고, gradle 환경에서는 필요 없다고 합니다.

다시 본론으로 돌아오면, intellij에서 file - settings - JAVA Compiler 에서 -paremeters 설정을 붙여주면 된다고 합니다.

자세한 내용은 https://velog.io/@appti/RequestBody-%ED%94%BC%EB%93%9C%EB%B0%B1%EC%9D%84-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B3%A0-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0-%EC%9C%84%ED%95%9C-%EA%B3%BC%EC%A0%95 참고

```java
@Getter
@AllArgsConstructor
public class RequestBodyDto {
    private String name;
    private int age;
}
```

해당 설정을 해준 뒤 위와 같이 기본 생성자가 아닌 전체 필드를 사용하는 생성자를 추가하는 @AllArgsConstructor 으로 변경하고 호출하면 정상적으로 객체가 생성되고, 필드 값들도 정상적으로 초기화가 됩니다.
