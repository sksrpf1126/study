# **_save() 주의점_**

해당 내용은 실전! 스프링 데이터 JPA(김영한님)의 강의를 보고 정리한 내용입니다.

---

Spring Data JPA에서 지원해주는 save 메서드를 사용할 때에는 주의해야 할 점이 존재한다.

```java
	@Transactional
	@Override
	public <S extends T> S save(S entity) {

		Assert.notNull(entity, "Entity must not be null.");

		if (entityInformation.isNew(entity)) {
			em.persist(entity);
			return entity;
		} else {
			return em.merge(entity);
		}
	}
```

내부 구조이며, 새로운 엔티티의 경우에는 em.persist()를 호출하며, 아닌 경우에는 em.merge를 통해 병합을 한다.

새로운 엔티티인지 아닌지는 엔티티의 식별자(@Id)를 기준으로 판별한다.

판단하는 기준은

- @Id가 객체일 때 null인지 아닌지로 판단
- 식별자가 자가 기본 타입(int, long과 같은 원시 타입)일 때에는 0인지 아닌지 판단(null값을 저장할 수 없으므로)
- Persistable 인터페이스를 구현해서 판단 메서드를 기준으로 판단

</br>

```java
    @Id
    @GeneratedValue
    private Long id;
```

위와 같이 @GeneratedValue가 설정되어 있으면, 식별자를 persist 동작 이후에 가져와서 세팅하기 때문에 save()는 정상적으로 작동하게 된다.

```java
    @Id
    private String id;
```

반대로, 위와같이 String타입을 선언하고 생성자로 id값을 미리 세팅하고 save()를 호출해버리면, 내부적으로 식별자 값이 존재하는것으로 판단하여 persist가 아닌 merge 메서드가 동작하게 되서, 추가적으로 쿼리가 발생하는 등 의도하지 않은 방향으로 실행이 된다.

그렇기에 @GeneratedValue를 사용하는게 편하며, 만약 Id를 프로젝트에서 정한 방향에 의해 다른 방법으로 해야한다면(위와 같이 직접 Id를 넣어주는 방법 등) Persistable 인터페이스를 구현해서 해결하면 된다.

</br>

```java
@Entity
//@Getter
@EntityListeners(AuditingEntityListener.class)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Item implements Persistable<String> {

    @Id
//    @GeneratedValue
    private String id;

    @CreatedDate
    private LocalDateTime createdDate;


    public Item(String id) {
        this.id = id;
    }

    @Override
    public String getId() {
        return null;
    }

    @Override
    public boolean isNew() {
        //createdDate가 null이면 persist이전이므로, JPA save 메서드에서 merge가 아닌 persist를 실행
        return createdDate == null;
    }
}
```

위와 같이 엔티티에 해당 인터페이스를 구현시키도록 하면 되며, 제네릭 타입은 id의 타입을 넣어주면 된다.

getId의 경우에는 내부적으로 메서드가 있어서, 오버라이딩을 해주었으며 핵심은 isNew 메서드이다. 바로 isNew메서드가 새로운 엔티티인지 아닌지를 판단하는 기준이 된다.

김영한님은 간단하게는 위와 같이 대부분의 테이블에는 엔티티의 생성 날짜를 저장하는 컬럼이 있으므로, @CreatedDate를 활용하여 isNew에 해당 변수가 null인지 아닌지로 판단한다고 하신다.
