# **_Querydsl 중급 문법_**

해당 내용은 실전! Querydsl(김영한님)의 강의를 보고 정리한 내용입니다.

---

## **_프로젝션_**

프로젝션이란 select 대상을 의미하며, 하나일 때와 둘일 때 반환타입이 달라진다.

### **_하나일 때_**

```java
/**
  * 프로젝션이 하나인 경우 해당 타입으로 받을 수 있음
  * select(member) -> 이것 또한 하나의 프로젝션이므로, Member타입으로 받을 수 있음
  */
@Test
public void simpleProjection() {

    List<String> result = queryFactory
            .select(member.username)
            .from(member)
            .fetch();
}
```

</br>

### **_둘 이상일 때_**

```java
/**
  * 프로젝션이 둘 이상인 경우
  * Tuple은 Querydsl에 종속적이므로, Service 꼐층으로 반환하는 것은 좋지 않음. 의존적이게 됨 그렇기에 DTO로 변환해서 반환하는 것을 권장
  */
@Test
public void tupleProjection() {
    //List<Member>는 불가능
    List<Tuple> result = queryFactory
            .select(member.username, member.age)
            .from(member)
            .fetch();

}
```

Tuple은 Querydsl에서 지원해주는 클래스로, 프로젝션이 여러개 일 때 Tuple로 반환타입이 사용 된다.

</br>

---

## **_DTO로 반환_**

DTO로 반환값을 받는 방법은 4가지 방법이 존재한다.

1. 프로퍼티 접근 방법 - setter를 활용
2. 필드 접근 방법
3. 생성자 방법
4. 생성자 + @QueryProjection 방법

### **_프로퍼티 접근 방법_**

```java
/**
  * 프로퍼티 접근 - setter를 활용한 DTO (기본 생성자 필수)
  */
@Test
public void findDtoBySetter() {
    List<MemberDto> result = queryFactory
            .select(
                    Projections.bean(MemberDto.class, member.username, member.age)
            )
            .from(member)
            .fetch();

    for (MemberDto memberDto : result) {
        System.out.println("memberDto = " + memberDto);
    }
}
```

Projections.bean을 활용하는 방법이다.

</br>

### **_필드 접근 방법_**

```java
/**
  * 필드 접근을 활용한 DTO - Setter필요 X
  */
@Test
public void findDtoByField() {
    List<MemberDto> result = queryFactory
            .select(
                    Projections.fields(MemberDto.class, member.username, member.age)
            )
            .from(member)
            .fetch();

    for (MemberDto memberDto : result) {
        System.out.println("memberDto = " + memberDto);
    }
}
```

Projections.fields를 활용하는 방법이다.

</br>

### **_생성자 방법_**

```java
/**
  * 생성자를 활용한 DTO (생성자 파라미터와 일치해야함)
  */
@Test
public void findDtoByConstructor() {
    List<MemberDto> result = queryFactory
            .select(
                    Projections.constructor(MemberDto.class, member.username, member.age)
            )
            .from(member)
            .fetch();

    for (MemberDto memberDto : result) {
        System.out.println("memberDto = " + memberDto);
    }
}
```

Projections.constructor를 활용한 방법이다.

</br>

### **_생성자 + @QueryProjection 방법_**

```java
/**
     * 생성자 + @QueryProjection 사용해서 DTO 반환
     * 위 constructor 방법과 다른 점은 위 생성자만 사용하는 방법은 member.id를 추가해버리면 일치하는 생성자가 존재하지 않지만, 컴파일 오류가 발생하지 않고
     * 실행 시점(런타임 시점)에 되어서야 에러가 발생한다.
     * 하지만 아래의 방법은 생성자에 존재하지 않는 파라미터를 매개변수로 넘길려하면 컴파일 에러 발생
     *
     * 단점 1. DTO에 대한 QType 클래스가 별도로 생성된다는 단점
     * 단점 2. QType을 생성하기 위해 생성자에 @QueryProjection를 붙임 -> 해당 DTO 클래스가 Querydsl에 의존적이게 됨
     *
     */
    @Test
    public void findDtoByQueryProjection() {
        List<MemberDto> result = queryFactory
                .select(new QMemberDto(member.username, member.age))
                .from(member)
                .fetch();

        for (MemberDto memberDto : result) {
            System.out.println("memberDto = " + memberDto);
        }
    }
```

단점이 존재하지만, 진행하는 프로젝트가 이미 Querydsl을 적극적으로 활용중이라면, 해당 방법을 권장한다.  
김영한님 또한 해당 방법을 사용하신다고 한다.

</br>

---

## **_동적 쿼리_**

동적 쿼리를 해결하는 방법은 2가지가 존재한다.

- BooleanBuilder 활용
- where 다중 파라미터 활용

### **_BooleanBuilder 방법_**

```java
private List<Member> searchMember1(String usernameCond, Integer ageCond) {

    //new BooleanBuilder(member.username.eq(usernameCond)); <= 초기값을 설정할 수 있음
    BooleanBuilder builder = new BooleanBuilder();
    if(usernameCond != null) {
        builder.and(member.username.eq(usernameCond));
    }

    if(ageCond != null) {
        builder.and(member.age.eq(ageCond));
    }

    return queryFactory
            .selectFrom(member)
            .where(builder)
            .fetch();
}
```

위와 같이 BooleanBuilder 객체를 만들어서 메서드를 통해 where절에 명시할 조건들을 넣어두고, 이후 쿼리의 where절에 해당 객체만 넣어주면 된다.

</br>

### **_where 다중 파라미터_**

```java
/**
  * where절에 null이 들어가면 무시
  * 쿼리안에 조건이 들어 있어 가독성 향상 (BooleanBuilder는 우선 처리 후에 쿼리가 발생)
  * 메서드로 분리함으로써, 조건을 재사용할 수 있음
  */
private List<Member> searchMember2(String usernameCond, Integer ageCond) {
    return  queryFactory
            .selectFrom(member)
            .where(usernameEq(usernameCond), ageEq(ageCond))
            .fetch();
}

//반환타입이 Predicate로 만들어지는데, BooleanExpression로 변경하는걸 권장(다른 조건메서드에서 재사용할 때 조립가능)
private BooleanExpression usernameEq(String usernameCond) {
    if(usernameCond == null) {
        return null; //무시
    }
    return member.username.eq(usernameCond);
}

private BooleanExpression ageEq(Integer ageCond) {
    if(ageCond == null) {
        return null;
    }
    return member.age.eq(ageCond);
}
```

위와 같이 where절에 들어갈 조건을 메서드로 추출하여서 해당 메서드마다 조건에 따라 QueryDsl의 쿼리를 반환하면 된다.

null로 반환시에는 해당 메서드는 조건으로 간주하지 않는다.

Predicate로 반환타입이 기본으로 정해지는데, BooleanExpression 타입으로 바꿔서 사용하자. 이유는 해당 메서드를 다른 메서드에서 조립할 때도 사용하기 위함이다.

```java
//조립
private BooleanExpression allEq(String usernameCond, Integer ageCond) {
    return usernameEq(usernameCond).and(ageEq(ageCond));
}
```

위와 같이 다른 메서드들을 재사용해서 조립해서 사용할 수 있다.

</br>

---

## **_벌크 연산_**

```java
/**
  * 벌크 연산은 항상 DB에 바로 쿼리를 날리므로, 영속성 컨텍스트의 캐시에 남아있는 데이터와 불일치 발생하므로, 연산 이후 영속성 컨텍스트를 비워주자.
  */
@Test
@Commit
public void bulkUpdate() {

    //member1, member2 => 비회원으로 변경
    //member의 나이를 +1 하는 문법은 set(member.age, member.age.add(1)) 와 같이 대상 컬럼, 바뀔 컬럼(값)을 적어준다.
    long count = queryFactory
            .update(member)
            .set(member.username, "비회원")
            .where(member.age.lt(28))
            .execute();
}
```

</br>

### **_delete 벌크 연산_**

```java
queryFactory
.delete(member)
.where(member.age.gt(18))
.execute();
```
