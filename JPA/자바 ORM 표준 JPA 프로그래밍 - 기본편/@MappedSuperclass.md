# **_@MappedSuperclass_**

해당 내용은 자바 ORM 표준 JPA 프로그래밍 - 기본편(김영한님)의 강의를 보고 정리한 내용입니다.

---

해당 애노테이션은 상속관계와 상관없이 여러 엔티티간에 공통의 필드(테이블간에 공통 컬럼들)들을 매핑할 때 사용한다.

```java
@MappedSuperclass
public abstract class BaseEntity {

    private String createdBy;
    private LocalDateTime createdDate;
    private String lastModifiedBy;
    private LocalDateTime lastModifiedDate;
}
```

와 같이 공통 필드들(공통 컬럼들)을 만들고, 클래스에 애노테이션을 걸어둔다.  
추상 클래스로 지정한 이유는 공통 매핑들만을 따로 사용하는 경우가 없기 때문이다.

다음으로, 공통 필드들을 사용하는 엔티티에다가 상속만 해주면 된다.

```java
public class Member extends BaseEntity
```

주의 해야할 것은 BaseEntity는 엔티티가 아니다. 즉, 테이블과 매핑하지 않는다는 것이다.

상속관계 매핑과 차이점을 꼽자면 상속관계 매핑은 어떠한 전략을 사용하든 결국 테이블로 구성을 하며, 그렇기에 엔티티로 관리가 된다.

하지만 BaseEntity의 경우에는 말 그대로 공통으로 사용할 필드들의 매핑 정보만을 담을 뿐 그에 대한 테이블이 만들어진다던가 관계가 맺어진다던가 하는 것이 아니다!

그렇기에 당연히 em.find(BaseEntity.class) 와 같은 검색이나 조회가 불가능 하다.
