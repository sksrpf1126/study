# MyBatis 환경에서의 DTO 분리

## 참고

https://tecoble.techcourse.co.kr/post/2021-04-25-dto-layer-scope/ -> 우테코 기술블로그 (DTO의 사용범위에 대하여)  
https://blog.javabom.com/minhee/spring-boot/date -> DTO Entity 분리와 분리하지 않은 모델의 장단점  
https://jaehhh.tistory.com/70 -> ResponseDTO와 RequestDTO 그리고 Domain 모델간의 분리를 해야하는가  
https://www.inflearn.com/questions/1044229/mybatis%EB%A7%8C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EA%B2%BD%EC%9A%B0%EC%9D%98-data-class-%EA%B5%AC%EC%A1%B0%EC%97%90-%EC%A1%B0%EC%96%B8-%EA%B5%AC%ED%95%A9%EB%8B%88%EB%8B%A4
-> 인프런 - MyBatis Data Class 구조 질문  
https://okky.kr/questions/986721 -> OKKY DTO 생성 및 분리에 대한 글  
https://www.inflearn.com/questions/374147/%EC%A7%88%EB%AC%B8%EC%9D%B4-%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4
-> 인프런 - Mybatis 환경에서도 엔티티와 DTO 변환을 통해 계층을 분리시켜야 하는가?  
https://umbum.dev/1206/ -> DTO와 Domain Model을 분리해야 하는 필요성

</br>

---

## DTO(Data Transfer Object)

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/2bf3b8b5-b4fa-4f1e-9ab2-e73eaed10cf2" width = 100%>
</p>

위 Presentation Layer를 Controller로 봐도 무방하며, Business Layer는 Service로, Data Access Layer를 Repository(Mybatis에서는 DAO)로 보면 됩니다.

그리고 위 계층간에 데이터 교환을 위해 사용하는 객체를 **_DTO_** 라고 합니다.

</br>

---

## MVC 패턴

<p align = "center" style = 'background-color : #ffffff'>
<img src="https://github.com/sksrpf1126/study/assets/62879192/44e6e6d5-e74d-4dc0-96b0-b0ce00a5eb96"  width = 100%>
</p>

MVC 패턴은 어플리케이션을 개발할 때 그 구성 요소를 Model과 View 및 Controller 등 세 가지 역할로 구분하는 디자인 패턴입니다. 비즈니스 처리 로직(Model)과 UI 영역(View)은 서로의 존재를 인지하지 못하고, Controller가 중간에서 Model과 View의 연결을 담당합니다.

라고 보통 MVC패턴을 설명합니다.

위 문장에서의 DTO를 분리하는 이유가 등장합니다.  
"비즈니스 처리 로직과 UI 영역은 서로의 존재를 인지하지 못하고, Controller가 중간에서 연결을 담당한다." 즉, Model영역과 View영역에서 사용하는 데이터 객체를 공유하는 행위는 서로의 존재를 인식시키는 것과 같다고 볼 수 있습니다.

말로는 와닿지 않기에 [우테코 DTO 분리에 대하여](https://tecoble.techcourse.co.kr/post/2021-04-25-dto-layer-scope/) 에서 사용하는 예시를 그대로 사용하겠습니다.

### User

```java
public class User {

    public Long id;
    public String name;
    public String email;
    public String password; //외부에 노출되서는 안 될 정보
    public DetailInformation detailInformation; //외부에 노출되서는 안 될 정보

    //비즈니스 로직, getter, setter 등 생략
}
```

### UserController

```java
@GetMapping
public ResponseEntity<User> showArticle(@PathVariable long id) {
    User user = userService.findById(id);
    return ResponseEntity.ok().body(user);
}
```

User라는 객체는 Model계층에서 사용하는 객체입니다. 이를 만약에 Controller에서 그대로 View로 반환하게 된다면 노출시키면 안되는 필드들 또한 그대로 전달하게 됩니다.

Model에서 사용하는 모든 속성이 외부에 노출되며, View에서 또한 사용하지 않는 필드들을 전달받게 되버립니다. 그리고 노출된 속성들은 보안 문제로 이어집니다.

또한, 만약에 User내부에 View에서는 사용하지 않는 필드들이 추가가 될 때마다 View 까지 그대로 전달되어 노출됩니다.

이러한 문제점들을 해결하기 위해서 계층(View - Controller - Model) 간에 DTO라는 객체로 한번 변환하여 View가 필요로 하는 필드들만 전달하여 사용할 수 있게 합니다.

### UserDTO

```java
public class UserDto {

    public final long id;
    public final String name;
    public final String email;

    //생성자 생략

    public static UserDto from(User user) {
        return new UserDto(user.getId(), user.getName(), user.getEmail());
    }
}
```

### UserController

```java
@GetMapping
public ResponseEntity<UserDto> showArticle(@PathVariable long id) {
    User user = userService.findById(id);
    return ResponseEntity.ok().body(UserDto.from(user));
}
```

위와 같이 비밀번호와 노출되면 안되는 정보는 DTO로 변환하면서 속성에서 제외했기 떄문에 앞에서 말한 문제점들을 보완할 수 있게 되었습니다.

위는 Model -> Controller -> View와 같은 응답(Response)을 예시로 들었지만, 요청(Request) 즉, View -> Controller -> Model로 요청을 처리하는 부분에서도 분리하는 경우가 일반적입니다.

그래서 보통 요청을 처리하는 DTO를 **_RequestDTO_** 라고 하며, 위와 같이 응답을 처리하기 위해 변환 대상이 되는 DTO를 **_ResponseDTO_** 라고 하여 각각의 클래스를 만들어서 사용합니다.

</br>

## DTO를 분리시키는 것이 무조건 좋은 것인가?

JPA 방식이라면 분리를 하여 Entity(DB의 테이블과 매핑되는 객체)를 DDD(도메인 주도 설계)방식으로 개발하는게 일반적이기 때문에 거의 대부분은 무조건 분리하는 방향으로 갑니다.  
문제는 사용해야 하는 기술이 MyBatis의 경우입니다.

### MyBatis

MyBatis는 JPA와는 다르게 Entity를 직접 DB와 매핑하고 다루는 기술은 존재하지 않기에, View에서 전달받은 데이터를 DTO객체에 담아 Entity객체로 변환하지 않고 바로 XML에 작성한 쿼리문을 통해 데이터를 받아와 객체로 만드는 경우가 많습니다.

즉, View - Controller - Model 구조 계층에서 하나의 DTO만으로 해결하는 방식을 많이 사용하게 됩니다.  
이를 통한 이점은 굳이 번거로운 변환과정을 거치지 않기 떄문에 빠른 속도로 개발할 수 있다는 것입니다.

그리고 이처럼 모든 계층에서 하나의 클래스로만 사용하는 것을 아래와 같이 **_"God Class"_** 라고 부릅니다.

<p align = "center" style = 'background-color : #ffffff'>
<img src="https://github.com/sksrpf1126/study/assets/62879192/02604a35-3d4d-4fa5-a701-56215800383a"  width = 100%>
</p>

전지전능한 신과 같이 하나의 클래스가 모든 계층에서 사용하여 처리하기에 이런 이름을 붙인 것 같습니다.

정리하면 변환 과정없이 하나의 클래스로 모든 계층에서 사용하게 된다면

1. 계층간 번거로운 변환 과정을 거치지 않아도 된다.
2. 변환 대상이 되는 클래스를 만들지 않아도 되기 때문에 적은 클래스로 프로젝트를 개발할 수 있다.

정도인 것 같습니다.

</br>

---

## 내가 생각한 구조

많은 글들을 읽고 고민을 하다가 https://jaehhh.tistory.com/70 글을 읽으며 해당 방법으로 가는것이 좋아보였습니다.

요청을 받아서 처리하는 ReuquestDTO를 Model에서 사용하기 위해 Entity 또는 VO 객체로 다시 변환하여 사용하는 것은 오히려 번거로운 작업이 될 수 있다고 합니다.  
그 이유는 요청으로부터 온 정보들은 대부분 그대로 사용하기 때문에 필요한 경우가 아닌 이상 그대로 사용해도 괜찮다는 것입니다.

반대로 응답의 경우에는 맨 위에서 설명한 User와 UserController의 문제점을 유발하지 않을 수 있어 항상 분리하는 것이 좋다고 합니다.

정리하면

### 요청의 경우

View -> Controller(RequestDTO로 받음) -> Service(특별히 따로 로직으로 처리하여 쿼리문으로 전달하는 것이 아니라면 그대로 사용) -> Repository or DAO (DTO의 정보를 토대로 쿼리 실행)

의 흐름이 될 것 같습니다.

</br>

### 응답의 경우

Repository or DAO(서비스 계층에서만 사용할 Entity 객체 또는 VO객체로 반환) -> Service(필요한 로직을 처리한 후 ResponseDTO로 변환하여 반환) -> Controller(ResponseDTO를 View로 그대로 반환) -> View(ResponseDTO를 통해 페이지 렌더링)

의 흐름이 될 것 같습니다.
