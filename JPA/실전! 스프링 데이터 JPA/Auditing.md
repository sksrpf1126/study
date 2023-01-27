# **_Auditing_**

해당 내용은 실전! 스프링 데이터 JPA(김영한님)의 강의를 보고 정리한 내용입니다.

---

## **_Auditing이란?_**

엔티티를 생성하거나 변경할 때 해당 행위를 한 사람이나 시간을 추적하고 싶을 때 사용하는 방법이다.  
보통 등록일, 수정일, 등록자, 수정자를 추적할 수 있다.

</br>

---

## **_순수 JPA 방법_**

순수 JPA로 Auditing을 사용해 보자.

```java
@MappedSuperclass
@Getter
public class JpaBaseEntity {

    @Column(updatable = false)
    private LocalDateTime createdDate;
    private LocalDateTime updatedDate;

    //저장하기 전 발생
    @PrePersist
    public void prePersist() {
        LocalDateTime now = LocalDateTime.now();
        createdDate = now;
        updatedDate = now;
    }

    //업데이트 전 발생
    @PreUpdate
    public void preUpdate() {
        updatedDate = LocalDateTime.now();
    }

}
```

@PrePersist는 저장되기 직전에 호출되고, @PreUpdate는 업데이트 전에 호출된다.  
이 외에도 @PostPersist, @PostUpdate가 존재한다.

그리고 추적할 엔티티에 상속시키면 되며, @MappedSuperclass를 꼭 지정해줘야 한다. 그래야 테이블의 컬럼에 해당 속성들이 추가되어 들어간다.

등록자와 수정자는 어떻게 추가하며, 컬럼은 어떻게 생기는지는 spring data jpa에서 보도록 하자.

</br>

---

## **_spring data jpa 방법_**

```java
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
@Getter
public class BaseEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;

}
```

위와 같이 애노테이션으로 사용이 가능하며, @EntityListeners 애노테이션이 있어야 한다. 글로벌로 적용이 가능하며, xml파일을 활용해야 한다.

추가적으로

```java
@EnableJpaAuditing
@SpringBootApplication
public class DataJpaApplication {

	public static void main(String[] args) {
		SpringApplication.run(DataJpaApplication.class, args);
	}


	@Bean
	public AuditorAware<String> auditorProvider() {
		return () -> Optional.of(UUID.randomUUID().toString());
	}

}
```

@EnableJpaAuditing을 루트 Application에 추가해줘야하며, 아래 @Bean 등록하는 부분은 등록자, 수정자를 UUID방식으로 추적하기 위해 추가한 것이며, 만약 spring security나 세션 쿠키 방식으로 사용한다면 각 방법에 맞게 return 해주면 된다.

마지막으로 적용할 엔티티에 BaseEntity 클래스를 상속시키면 된다.

</br>

```java
@Test
public void JpaEventbaseEntity() throws Exception {

  Member member = new Member("member1");
  memberRepository.save(member); // @PrePersist 동작

  Thread.sleep(100);
  member.setUsername("member2");

  em.flush(); //@PreUpdate 동작
  em.clear();

  Member findMember = memberRepository.findById(member.getId()).get();

  System.out.println("findMember.createdDate = " + findMember.getCreatedDate());
  System.out.println("findMember.updatedDate = " + findMember.getLastModifiedDate());
  System.out.println("findMember.createdBy = " + findMember.getCreatedBy());
  System.out.println("findMember.updatedBy = " + findMember.getLastModifiedBy());

}
```

위와 같이 테스트 코드를 실행하면 아래와 같이 DB에 컬럼이 생겨서 저장하게 된다.

</br>

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/214291430-43d90579-a81c-4f47-9ed6-c175d295273a.png" width = 100%>
</p>
