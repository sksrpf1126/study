# **_Querydsl 기본 문법_**

해당 내용은 실전! Querydsl(김영한님)의 강의를 보고 정리한 내용입니다.

---

## **_QType_**

Querydsl은 엔티티에 대한 QType을 만들어서, 해당 클래스를 활용해서 쿼리를 작성한다.

QType은 각 엔티티마다 만들어지며 빌드를 할 때, 지정해 놓은 위치에 생긴다.

```java
    @Test
    public void startQuerydsl() {
//        QMember m = new QMember("m");
        QMember m = QMember.member;

        Member findMember = queryFactory
                .select(m)
                .from(m)
                .where(m.username.eq("member1"))
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
```

위와 같이 new 연산자를 통해 객체를 만들어서 생성자에 별칭을 지정해서 사용해도 되며, QMember.member와 같이 사용해도 된다.

QMember.meber는 QMember 클래스에 아래와 같이 필드가 존재하기 때문에 사용이 가능한 것이다.

```java
    public static final QMember member = new QMember("member1");
```

마지막은 QMember를 static import를 하여서 사용하는 방법으로, 제일 권장하는 방법이라고 하신다.

```java
    @Test
    public void startQuerydsl2() {

        //member == QMember.member => static import
        Member findMember = queryFactory
                .select(member)
                .from(member)
                .where(member.username.eq("member1"))
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
```

</br>

---

## **_다양한 메서드_**

```java
member.username.eq("member1") // username = 'member1'
member.username.ne("member1") //username != 'member1'
member.username.eq("member1").not() // username != 'member1'
member.username.isNotNull() //이름이 is not null
member.age.in(10, 20) // age in (10,20)
member.age.notIn(10, 20) // age not in (10, 20)
member.age.between(10,30) //between 10, 30
member.age.goe(30) // age >= 30  Great(큰) Or (또는) Equal (같다)  즉, 30보다 크거나 같다.
member.age.gt(30) // age > 30 greate(큰) 즉, 30보다 크다.
member.age.loe(30) // age <= 30 Little(작다) Or (또는) Equal (같다) 즉, 30보다 작거나 같다.
member.age.lt(30) // age < 30 Little(작다) => 30보다 작다.
member.username.like("member%") //like 검색 => 전방 일치 검색
member.username.contains("member") // like ‘%member%’ 검색  =>중간 일치 검색
member.username.startsWith("member") //like ‘member%’ 검색 =>후방 일치 검색
```

위와 같이 SQL에서 사용할 수 있는 문법들을 Querydsl은 다양한 메서드로 구현해 놓았다. 더 다양한 메서드들이 존재하며, 자세한 내용은 문서를 참고하자.

</br>

---

## **_결과 조회 메서드_**

```java
결과 조회 메서드

- fetch() : 리스트 조회, 데이터 없으면 빈 리스트 반환
- fetchOne() : 단 건 조회
=>결과가 없으면 : null
=>결과가 둘 이상이면 : com.querydsl.core.NonUniqueResultException
- fetchFirst() : limit(1).fetchOne()
- fetchResults() : 페이징 정보 포함, total count 쿼리 추가 실행 -> deprecated가 되었으며, 공식문서에서는 그냥 fetch()를 추천하고 있다. (offset, limit를 활용)
- fetchCount() : count 쿼리로 변경해서 count 수 조회
```

</br>

---

## **_집합 연산_**

SQL에서는 count, max, min 등 여러 집합 연산을 제공해주는데, Querydsl 또한 이를 지원해준다.

```java
List<Tuple> result = queryFactory
        .select(
                member.count(),
                member.age.sum(),
                member.age.avg(),
                member.age.max(),
                member.age.min()
        )
        .from(member)
        .fetch();

//Tuple은 Querydsl에서 제공하는 클래스 =>실무에서는 DTO로 뽑아오는 방법을 더 많이 씀
Tuple tuple = result.get(0);

assertThat(tuple.get(member.count())).isEqualTo(4);
assertThat(tuple.get(member.age.sum())).isEqualTo(100);
assertThat(tuple.get(member.age.avg())).isEqualTo(25);
assertThat(tuple.get(member.age.max())).isEqualTo(40);
assertThat(tuple.get(member.age.min())).isEqualTo(10);
```

위와 같이 Tuple로 반환타입이 지정되는 이유는, select 결과가 하나의 엔티티의 데이터가 아닌 다양한 타입의 데이터를 반환하기 때문이다.

더 자세한 내용은 중급 문법에서 다루며, 보통 Tuple보다는 직접 DTO를 만들어서 사용하는 편이 더 많다고 한다.

</br>

---

## **_Group By_**

```java
List<Tuple> result = queryFactory
        .select(
                team.name,
                member.age.avg()
        )
        .from(member)
        .join(member.team, team)
        .groupBy(team.name)
        .fetch();
```

group by는 SQL 의 대표적인 문법 중 하나이기 때문에 당연히 Querydsl에서 사용이 가능하다.

</br>

---

## **_연관관계 없이 조인(세타 조인)_**

```java
//연관관계 없이 세타조인 즉, 크로스 조인해서 username == team.name 일치하는 것을 가져오는 것 없는 건 team 데이터는 null
//leftJoin(member.team, team)을 하면 속한 팀에서 유저이름이랑 팀 이름 같은지를 판별하지만
//아래와 같이 leftJoin(team)하면 막 조인을해서 연관관계 신경안쓰고 그냥 team이름과 username이 같은 레코드를 가져옴
List<Tuple> result = queryFactory
        .select(member, team)
        .from(member)
        .leftJoin(team)
        .on(member.username.eq(team.name))
        .fetch();
```

주석으로 내용을 정리했지만, 다시 한번 정리하면 Querydsl의 경우 Join문법을 사용할 때 join(대상이 되는 엔티티, 조인할 엔티티)와 같이 2개의 조인 대상을 넘겨줘야 하지만 이는 연관관계를 기준으로 조인할 때이며, 위는 연관관계 없이 조인하는 방법이다.

leftJoin을 team으로 두며 member의 username과 team의 name이 일치하는 결과를 반환한다.

</br>

---

## **_fetch join_**

JPQL에서는 join fetch문법을 사용해서 패치 조인을 했는데, Querydsl은 어떻게 사용해야 할까?

```java
 @Test
public void fetchJoinUse() throws Exception {
    em.flush();
    em.clear();

    Member findMember = queryFactory
            .selectFrom(member)
            .join(member.team, team).fetchJoin()
            .where(member.username.eq("member1"))
            .fetchOne();
    boolean loaded =
            emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
    assertThat(loaded).as("페치 조인 미적용").isTrue();
}
```

위와 같이 join메서드 이후에 fetchJoin을 붙여주기만 하면 패치 조인이 이루어진다.

</br>

---

## **_서브 쿼리_**

```java
/**
  * 나이가 가장 많은 회원 조회
  */
@Test
public void subQuery() {

    QMember memberSub = new QMember("memberSub");

    List<Member> result = queryFactory
            .selectFrom(member)
            .where(member.age.eq(
                    JPAExpressions
                            .select(memberSub.age.max())
                            .from(memberSub)
            ))
            .fetch();

    assertThat(result).extracting("age")
            .containsExactly(40);
}
```

서브 쿼리는 JPAExpressions 를 활용하여 작성하며, 주의해야할 점은 select절과 where절에서는 서브쿼리가 가능하지만 From(인라인 뷰)절에서는 사용이 불가능하다.

그렇기에 만약 From절에 서브쿼리를 사용해야할꺼 같을 때 Join으로 풀 수 있는지를 확인하고, 안되면 쿼리를 분할해서 하거나, Native Query를 사용해야 한다.

</br>

---

## **_변수 사용_**

```java
@Test
public void constant() {
    List<Tuple> result = queryFactory
            .select(member.username, Expressions.constant("A"))
            .from(member)
            .fetch();

    for (Tuple tuple : result) {
        System.out.println("tuple = " + tuple);
    }
}
```

위와 같이 Expressions를 활용하면 반환 결과에 변수의 값을 반환 값으로 받을 수 있다.  
위는 "A"를 반환하며, Tuple로 반환하는 이유는 Member엔티티 뿐만 아니라 "A"의 데이터도 반환하기 때문에 여러 타입에 의해 Tuple로 반환하는 것이다.

쿼리에서는 "A"와 관련된 쿼리는 나가지 않으며, 먼저 쿼리를 통해 member.username을 가져오고 이후에 "A"를 처리하는 것이다.

### **_활용 예제_**

```java
NumberExpression<Integer> rankPath = new CaseBuilder()
.when(member.age.between(0, 20)).then(2)
.when(member.age.between(21, 30)).then(1)
.otherwise(3);

List<Tuple> result = queryFactory
.select(member.username, member.age, rankPath)
.from(member)
.orderBy(rankPath.desc())
.fetch();
```

21 ~ 30세의 나이를 가진 member를 우선적으로 반환하고, 이후에는 0 ~ 20세를 마지막은 나머지를 순서대로 반환하는 쿼리이다.

Querydsl은 자바 코드로 작성하기 때문에 rankPath 처럼 복잡한 조건을 변수로 선언해서 select 절, orderBy 절에서 함께 사용할 수 있다.
