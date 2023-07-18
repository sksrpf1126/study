# **_QueryDSL을 통한 동적 쿼리 개선_**

해당 내용은 [사이드 이펙트](https://github.com/Side-Effect-Team/side-effect-backend) 프로젝트와 관련된 내용입니다.

---

## **_배경_**

저가 진행하고 있는 프로젝트에서는 처음에 바로 QueryDSL을 도입하지 않고, JPQL과 Spring Data JPA로 개발을 해보고 필요성을 느끼면 그때 QueryDSL을 사용하기로 회의를 통해 정했었습니다.

그러다가 개발을 하던 중에 **_동적 쿼리_** 를 작성하는 경우가 생겼습니다.  
해당 경우는 모집 게시판의 게시글을 검색을 할 때 사용해야 하는 쿼리로 No Offset 방식의 페이징 처리와 함께 검색어 그리고 기술 스택들을 검색 조건으로 사용하게 됩니다.  
즉, 검색어(keyword)나 기술 스택(stackTypes)이 존재하는 경우에만 where 구문의 조건에 쓰이도록 동적 쿼리를 작성해야 하는 상황이었습니다.

그래서 우선 JPQL방식으로 동적 쿼리를 작성하고, 이후에 QueryDSL이 도입되면 개선하기로 하였습니다.

---

## **_JPQL방식의 동적 쿼리_**

QueryDSL을 도입하기 전의 동적 쿼리는 다음과 같습니다.

```java
    @Override
    public List<RecruitBoard> findWithSearchConditions(Long lastId, String keyword, List<StackType> stackTypes, Pageable pageable) {

        String jpql = "select distinct rb from RecruitBoard rb";
        String joinSql = "";
        String whereSql = " where ";
        List<String> whereConditions = new ArrayList<>();

        if(lastId != null) {
            whereConditions.add("rb.id < :lastId");
        }

        if(stackTypes != null && !stackTypes.isEmpty()) {
            joinSql += " join rb.boardStacks bs";
            whereConditions.add("bs.stack.stackType in :stackTypes");
        }

        if(StringUtils.hasText(keyword)) {
            whereConditions.add("(rb.title like concat('%',:keyword,'%') or rb.contents like concat('%',:keyword,'%'))");
        }

        if(StringUtils.hasText(joinSql)) {
            jpql += joinSql;
        }

        if(!whereConditions.isEmpty()) {
            jpql += whereSql;
            jpql += String.join(" and ", whereConditions);
        }

        jpql += " order by rb.id desc";

        TypedQuery<RecruitBoard> query = em.createQuery(jpql, RecruitBoard.class);

        if(lastId != null) {
            query.setParameter("lastId", lastId);
        }

        if(stackTypes != null && !stackTypes.isEmpty()) {
            query.setParameter("stackTypes", stackTypes);
        }

        if(StringUtils.hasText(keyword)) {
            query.setParameter("keyword", keyword);
        }

        return query.setMaxResults(pageable.getPageSize()).getResultList();
    }
```

No Offset 방식의 페이징 처리이기 때문에 lastId 여부에 따라 쿼리를 추가해야하며, 이 외에도 검색어(keyword)나 기술 스택(stackTypes)의 여부에 따라 쿼리를 추가하여 최종적으로 조건에 맞는 게시글들을 요청한 size 수만큼 반환하도록 합니다.

코드는 엄청 길어졌으며, 쿼리 자체를 문자열로 만들어서 붙여 최종적으로 사용하다 보니, 문자열에 오타가 나도 컴파일 시점에서는 에러를 찾을 수 없다는 문제점을 가지고 있습니다.

위 코드보다 더 간결하게 작성할 수는 있겠지만 JPQL 방식을 계속 사용한다면 극적으로 개선될 수는 없을 것이라 생각합니다.

---

## **_QueryDSL방식의 동적 쿼리_**

다른 팀원분들도 JPQL 방식으로 작성하다가 동일한 문제에 직면하였으며, 추가적인 요청사항으로 로그인한 사용자가 좋아요를 누른 게시글들에 대해서는 True값을 아니라면 False값을 반환하는 상태값을 추가해야 되는 상황이였습니다.  
이러한 상황에서 JPQL을 작성한다면 코드는 더 길어질 것이며 가독성에 문제가 생길것이라 생각했습니다. 저와 팀원분들은 이런 문제점을 체감하였고 QueryDSL을 도입했습니다.

도입 이후에 QueryDSL로 리팩토링한 코드는 다음과 같습니다.

### **_게시판 검색조건 페이징 처리 메서드_**

```java
@Override
public List<RecruitBoardAndLikeDto> findWithSearchConditions(Long userId, Long lastId, String keyword, List<StackType> stackTypes, Integer size) {
    JPAQuery<RecruitBoardAndLikeDto> query = jpaQueryFactory.selectDistinct(getResponseConstructor(userId)).from(recruitBoard);

    if(stackTypes != null && !stackTypes.isEmpty()) {
        query.innerJoin(recruitBoard.boardStacks, boardStack).innerJoin(boardStack.stack, stack);
    }

    return query.where(lastIdLt(lastId), addKeywordCondition(keyword), addStackTypeCondition(stackTypes))
            .orderBy(recruitBoard.id.desc())
            .limit(size).fetch();
}
```

조회할 컬럼정보들을 getResponseConstructor 메서드로 분리하였으며, 검색조건으로 기술스택이 존재할 경우에만 조인을 하도록 했습니다.  
마지막으로, where조건들을 메서드로 각각 분리해서 사용하고 최종적으로 요청한 size 수만큼 조회하여 반환합니다.

각각에 사용되는 메서드들은 아래에 정리해 놓았습니다.

### **_SELECT 컬럼 메서드_**

```java
private ConstructorExpression<RecruitBoardAndLikeDto> getResponseConstructor(Long userId) {

    return Projections.constructor(RecruitBoardAndLikeDto.class,
            recruitBoard,
            getLikeExpression(userId)
    );
}

private Expression<Boolean> getLikeExpression(Long userId) {
    if (userId == null) {
        return Expressions.asBoolean(false).isTrue();
    }
    return ExpressionUtils.as(JPAExpressions.selectOne()
                    .from(recruitLike)
                    .where(recruitLike.user.id.eq(userId).and(recruitLike.recruitBoard.id.eq(recruitBoard.id))).limit(1).isNotNull(),
            "like");
}
```

조회할 컬럼들을 정의하는 메서드로, getResponseConstructor 메서드는 recruitBoard 엔티티에 담겨져있는 필드들과 getLikeExpression이라는 메서드가 반환하는 컬럼으로 반환합니다.

getLikeExpression은 로그인한 사용자가 게시글에 좋아요를 눌렀을 경우 True값을 이 외에 로그인을 하지 않았거나 좋아요를 누르지 않은 게시글인 경우에는 false값을 반환하도록 하는 메서드입니다.

### **_WHERE 검색 조건 메서드_**

```java
private BooleanExpression lastIdLt(Long lastId) {
    return lastId != null ? recruitBoard.id.lt(lastId) : null;
}

private BooleanExpression addKeywordCondition(String keyword) {
    return hasText(keyword) ? recruitBoard.title.containsIgnoreCase(keyword).or(recruitBoard.contents.containsIgnoreCase(keyword)) : null;
}

private BooleanExpression addStackTypeCondition(List<StackType> stackTypes) {
    if(stackTypes == null || stackTypes.isEmpty()) {
        return null;
    }

    return boardStack.stack.stackType.in(stackTypes);
}
```

마지막으로 WHERE 조건으로 들어가는 메서드들입니다.  
기존의 JPQL은 하나의 메서드에 복잡하게 if문으로 구분하면서 쿼리를 추가하였지만 QueryDSL은 각각의 조건들을 메서드로 추출하여 사용할 수 있습니다.

언뜻 보기에는 코드의 양에는 별 차이가 없어보입니다.  
하지만 메서드로 분리함으로써 가독성도 좋아졌을 뿐만 아니라 이후 다른 쿼리에서 재사용을 할 수 있다는 장점이 있습니다.  
또한, QueryDSL은 쿼리를 메서드 단위로 붙이면서 만들기 때문에 컴파일 시점에서 코드의 오타를 잡을 수 있습니다.

정리하면 QueryDSL은 JPQL에 **_유연성, 가독성, 재사용성_** 을 높여주는 도구(라이브러리) 입니다.
