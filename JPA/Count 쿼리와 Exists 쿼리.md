# Count 쿼리와 Exists 쿼리

사이드 프로젝트를 진행하면서 Spring Data JPA를 사용하게 되었는데, 유저가 해당 게시판의 포지션에 지원한 정보가 있다면, 해당 유저는 동일한 게시글의 어떠한 포지션이든 상관 없이 지원을 할 수 없도록 해야했다.  
이때, 지원한 정보(데이터)가 있는지를 판단하기 위해 쿼리를 생각하면서 Count 쿼리와 Exists 쿼리를 보통 사용한다고 하여, 둘에 차이가 있는지 찾아보다가 아래의 글을 읽게 되었다.  
https://jojoldu.tistory.com/516

자세히 설명과 성능까지 비교한 글이다.

</br>

---

## **_이해한 내용 정리_**

위 블로그 글을 정리하면, count쿼리는 exists쿼리보다 성능이 좋지 않으며. 성능의 차이는 2배 가까이 차이가 난다고 한다. 그리고 이는 데이터가 많아지면 많아질수록 성능의 차이는 더 심해진다.

그러면 무슨 차이가 있길래, count 쿼리가 성능이 좋지 않을까?  
count쿼리는 조건으로 필터링 된 데이터들을 모두 카운팅을 해야한다. 즉, 조건으로 필터링 된 데이터가 많으면 많을수록 카운팅해야 할 데이터의 수도 그만큼 증가한다는 것이다.  
하지만, exists쿼리는 해당 조건에 만족하는 단 하나의 데이터만 존재한다면 true를 반환하고 리턴하게 된다. 즉, 데이터의 수를 세는게 아니라 값의 여부를 판단하기 위해서라면 전부 데이터를 센 후에 한 건이라도 있을 경우 반환하는 count쿼리 보다는, 단 하나라도 만족하면 중단하고 true를 반환하는 exists쿼리가 월등히 성능이 좋다는 것이다.

하지만, 무조건 exists가 성능이 좋다고는 할 수 없다. 조건으로 필터링 된 데이터가 매우 적다면? 그렇다면 전체적인 쿼리를 보고 판단해야 할수도 있다. 그리고 이는 이후에 설명할 프로젝트에서 exists쿼리가 아닌 count쿼리를 쓰는 이유가 되기도 한다. (실제로는 exists쿼리가 아닌 limit 1로 개선한 쿼리)

</br>

---

## **_JPA에서는 Exists쿼리를 어떻게?_**

해당 글을 정리한 핵심적인 이유이다. JPA에서는 다양한 방식으로 쿼리를 사용할 수 있는데, 대표적으로는 JPQL, spring data JPA, Querydsl로 나눠서 설명한다.

### **_JPQL_**

JPQL에서는 아쉽게도, select절에 exists쿼리를 사용할 수 없다. 물론 where절에는 사용이 가능하지만, where절에 추가적인 서브쿼리를 작성해야 사용할 수 있으며, 이는 매우 불편하다.  
하지만, Querydsl을 사용하면 해결할 수 있다. 정확히 말하면 Querydsl로 exists쿼리와 거의 동등한 성능을 낼 수 있도록 쿼리를 작성할 수 있다.

### **_Spring Data JPA_**

spring data JPA는 메서드명을 활용해서 쿼리를 생성하는 방식을 사용한다. 그리고 existsBy~~~와 같이 메서드명을 작성하면, 반환값을 boolean형태로 값의 여부를 판단할 수 있다.  
그러면, 실제로도 쿼리는 exists를 사용하게 되는 것일까?

맨 위의 블로그 글에 설명이 자세히 되어 있다.

```java
boolean existsByName(String name);
```

으로 해당 메서드를 사용하면 실제 쿼리는 아래와 같다.

```sql
select b.id
from book b
where b.name = ? limit 1
```

실제로는 exists쿼리가 아닌 조건에 맞는 데이터를 하나만 가져오도록 해서, exists와 동등한 성능을 내며, spring data JPA는 알아서 저렇게 쿼리를 날린다.  
하지만 spring data JPA는 쿼리가 복잡해지기 시작하면, 메서드명으로는 한계가 있거나 매우 길어져서 가독성이 떨어지게 된다는 단점이 있다. 그래서 위와 같이 간단한 쿼리정도에 사용하면 될 것 같다.

### **_Querydsl_**

결론부터 말하면 Querydsl로도 select exists 방식으로 사용할 수는 없다. (where절에는 사용이 가능)  
하지만 앞에 Spring data JPA에서 exists와 동등한 쿼리를 알 수 있었다. Querydsl도 위와 같이 limit 1을 통해 쿼리를 작성해서 날리면 될 뿐이다.

</br>

```
참고내용
 JPQL에는 limit 쿼리를 제공하지 않는다고 한다. 그래서 간단한 쿼리는 spring data JPA 방식으로 사용하고, 복잡해진다고 하면 Querydsl을 사용해야 한다.
```

</br>

---

## **_프로젝트에 사용한 쿼리_**

해당 내용을 정리하는 시점에서는 프로젝트에 아직 Querydsl을 도입하기 전이다. (JPQL과 spring data JPA로 우선 구현을 해보고, Querydsl의 필요성을 느끼면 도입하여 리팩토링할 예정)

그래서 우선 spring data JPA방식으로 메서드명을 통해 쿼리를 구현해 보았다.

```java
 boolean existsByIdAndBoardPositions_Applicants_UserId(Long boardId, Long userId)
```

게시판의 Id와 지원자의 Id를 받아서, 게시판에서 구하고 있는 모든 포지션의 지원자 목록에 지원자의 Id가 존재하는지를 판단하는 메서드이다.

실제 쿼리는 아래와 같이 나간다.

```sql
select
            recruitboa0_.recruit_board_id as col_0_0_
        from
            recruit_board recruitboa0_
        left outer join
            board_position boardposit1_
                on recruitboa0_.recruit_board_id=boardposit1_.recruit_board_id
        left outer join
            applicant applicants2_
                on boardposit1_.board_position_id=applicants2_.board_position_id
        left outer join
            users user3_
                on applicants2_.user_id=user3_.user_id
        where
            recruitboa0_.recruit_board_id=?
            and user3_.user_id=? limit ?
```

limit ?부분을 통해서, spring data JPA가 알아서 최적화된 쿼리를 보내는 것을 확인할 수 있었다.

하지만, applicant 테이블에 존재하는 user_id가 아닌, applcant 테이블과 user 테이블을 조인한 뒤의 user테이블의 id를 조건문으로 사용하게 된다. 즉, 불필요한 조인이 발생하는 것이다.

그래서 아래와 같이 JPQL 방식으로 쿼리를 변경을 해봤다.

```java
    @Query("SELECT CASE WHEN COUNT(a) > 0 THEN true ELSE false END " +
            "FROM RecruitBoard rb " +
            "JOIN rb.boardPositions bp " +
            "JOIN bp.applicants a " +
            "WHERE rb.id = :boardId " +
            "AND a.user.id = :userId")
    boolean  existsApplicantByRecruitBoard(@Param("boardId") Long boardId, @Param("userId") Long userId);
```

그리고 실제로 나가는 쿼리는 아래와 같다.

```sql
select
    case
        when count(applicants2_.applicant_id)>0 then 1
        else 0
        end as col_0_0_
from
    recruit_board recruitboa0_
inner join
    board_position boardposit1_
        on recruitboa0_.recruit_board_id=boardposit1_.recruit_board_id
inner join
    applicant applicants2_
        on boardposit1_.board_position_id=applicants2_.board_position_id
where
    recruitboa0_.recruit_board_id=?
    and applicants2_.user_id=?
```

count쿼리를 통해서 조회하지만, left outer join 3번이 inner join 2번으로 변경되었다.  
하나의 글에 무수히 많은 지원자들이 몰릴 경우에는 spring data JPA 방식의 limit 1 방식이 효율적일 수도 있겠으나, 그렇지 않는다면 user 테이블과의 JOIN이 오히려 성능이 악화될 것 같다고 판단하여 후자의 방법을 선택했다.

또한, spring data JPA 방식은 메서드 명이 너무 길어진다는 단점 또한 가지고 있다.

물론, Querydsl을 적용하면, count쿼리가 아닌 limit 1 방식으로 바꿔볼 예정이다.

</br>

---

## **_Querydsl 도입_**

JPQL만으로 동적쿼리를 작성하다보니 쿼리가 너무 복잡해져서 프로젝트에 Querydsl을 도입하기로 하였다.

이에 따라 count쿼리 또한 Querydsl을 통해 리팩토링을 하기로 하였다.

```java
    @Override
    public boolean existsApplicantByRecruitBoard(Long boardId, Long userId) {
        Integer fetchOne = jpaQueryFactory
                .selectOne()
                .from(recruitBoard)
                .innerJoin(recruitBoard.boardPositions, boardPosition)
                .innerJoin(boardPosition.applicants, applicant)
                .where(recruitBoard.id.eq(boardId), applicant.user.id.eq(userId))
                .fetchFirst();

        return fetchOne != null;
    }
```

위 방식이 기존의 JPQL로 작성한 Count쿼리를 Querydsl로 변경한 코드이다.

핵심은 마지막의 fetchFirst()로, 내부적으로 limit 1 이 붙은 쿼리가 나가서, 조건에 맞는 데이터가 있으면 이후의 필터링은 거치치 않고, 해당 데이터를 바로 반환하게 된다.  
이후, selectOne()을 통해 데이터가 존재하면 반환값으로 1을, 없다면 null을 반환하기 때문에 null이 아니라면 해당 게시글에 지원자가 이미 지원했음을 알 수 있다.

</br>

```sql
select
            1 as col_0_0_
        from
            recruit_board recruitboa0_
        inner join
            board_position boardposit1_
                on recruitboa0_.recruit_board_id=boardposit1_.recruit_board_id
        inner join
            applicant applicants2_
                on boardposit1_.board_position_id=applicants2_.board_position_id
        where
            recruitboa0_.recruit_board_id=?
            and applicants2_.user_id=? limit 1
```

위는 Querydsl로 변경 후에 실제로 나가는 쿼리이다.  
spring data JPA의 조인 3번이 나가는 문제도 해결하였으며, count쿼리의 문제점까지 해결이 되었다.

처음부터 바로 Querydsl을 도입하여 사용을 했었다면 JPQL, spring data JPA가 어떤 부분이 지원이 안되고, 어떻게 동작하는지 자세히 알지 못하고 넘어갔겠지만, 단계적으로 JPQL과 spring data JPA를 사용하다가 한계점을 느끼거나 복잡해질 때 Querydsl을 도입하다 보니 Querydsl을 왜 사용하는지 확실히 체감이 되었다.
