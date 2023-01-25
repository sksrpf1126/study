# **_spring data jpa 기본_**

해당 내용은 실전! 스프링 데이터 JPA(김영한님)의 강의를 보고 정리한 내용입니다.

---

## **_인터페이스 기반 레포지토리_**

spring data jpa는 인터페이스를 기반으로 레포지토리를 만든다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

}
```

위와 같이 인터페이스 기반으로 spring data jpa를 사용하며, @Repository가 없어도 빈으로 등록이 된다. 그 이유는 spring data jpa가 알아서 해주기 때문이다.

그런데 구현체가 없는 인터페이스가 어떻게 다른 곳에서 주입받아서 사용이 되는걸까?  
data jpa는 인터페이스 레포지토리를 기반으로 프록시 구현체를 생성하여 해당 프록시를 주입시켜서 사용한다.

</br>

---

## **_반환 타입과 조회 건수_**

spring data jpa를 통해 조회한 결과가 없다면 null을 넣어줄까? 두 가지 경우가 존재한다.

1. 반환타입이 컬렉션일 때  
   -> null이 아닌 빈 컬렉션을 반환한다.

2. 컬렉션이 아닐 때  
   -> null값을 반환한다.

하지만 null로 반환할 가능성이 있는 메서드의 경우에는 Optional을 적극 활용하자.

</br>

반대로, 단건 조회인데 조회 결과가 둘 이상이라면?  
==> javax.persistence.NonUniqueResultException 예외를 반환한다.

</br>

---

## **_벌크 연산_**

@Modifying 애노테이션은 JPQL의 벌크 연산 쿼리에서 마지막에 작성하는 excuteUpdate()를 해주는 애노테이션이라고 생각하면 된다.  
그렇기에 spring data jpa로 벌크 연산을 하는 경우 해당 애노테이션이 있어야 한다. 없으면 예외 반환

@Modifying(clearAutomatically = true)를 통해 벌크성 쿼리를 하고나서 영속성 컨텍스트를 초기화 해줄 수 있다. (default값은 false)  
true로 안해주면 find를 할 때 1차 캐시에서 가져와서 데이터 불일치가 발생할 수 있다.

벌크 연산은 영속성 컨텍스트를 무시하고 실행하기 때문에, 영속성 컨텍스트에 있는 엔티티의 상태와
DB에 엔티티 상태가 달라질 수 있다. 그렇기에 아래의 권장하는 방안을 사용하도록 하자.

> 권장하는 방안
>
> 1. 영속성 컨텍스트에 엔티티가 없는 상태에서 벌크 연산을 먼저 실행한다.
> 2. 부득이하게 영속성 컨텍스트에 엔티티가 있으면 벌크 연산 직후 영속성 컨텍스트를 초기화 한다.

</br>

---

## **_페이징 처리_**

아래와 같이 페이징의 방법과 슬라이스 방법이 존재한다. 슬라이스 방법은 우리가 아는 스크롤 내릴 때 더보기 버튼을 누를 때 추가로 가져온다던가, 무한 스크롤 방식으로 스크롤을 내릴 때 추가로 가져오는 방식에서 사용이 되며, 10개의 데이터를 보여줘야 한다면 10 + 1개를 더 가져오도록 해서 + 1개가 존재하면 더보기 버튼을 유지한다던지 하는 방향으로 설계할 수 있다.

```java
    //totalcount를 따로 쿼리를 작성해서 성능 최적화
    @Query(value = "select m from Member m left join m.team t", countQuery = "select count(m.username) from Member m")
    Page<Member> findByAge(int age, Pageable pageable);

    //find와 by사이 아무거나 가능
    Slice<Member> findSliceByAge(int age, Pageable pageable);
```

spring data jpa 페이징은 가져올 데이터들에 대한 쿼리뿐만 아니라 totalCount 쿼리도 날려서 총 개수를 가져온다.  
그런데 가져올 데이터들의 쿼리가 여러 조인이 들어가는 경우 totalCount 쿼리도 복잡해져서 성능이 저하가 될 가능성이 존재한다.  
이럴 떄는 @Query의 countQuery 속성을 통해 따로 totalCount를 가져오는 쿼리를 작성해주자.
