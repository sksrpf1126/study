# **_JPQL 중급 문법_**

해당 내용은 자바 ORM 표준 JPA 프로그래밍 - 기본편(김영한님)의 강의를 보고 정리한 내용입니다.

---

## **_경로 표현식_**

경로 표현식이란 JPQL에서 .(점)을 찍어 객체 그래프를 탐색하는 것이다.

경로 표현식에는 크게 3가지로 구분할 수 있다.

```java
select m.username -> "상태 필드"
from Member m
join m.team t -> 단일 값 "연관 필드"
join m .orders o -> 컬렉션 값 "연관 필드"
```

- 상태 필드  
  -> 상태 필드는 단순한 값을 저장하기 위한 필드로 엔티티 객체에서 연관관계가 존재하지 않는 단순한 값 필드들을 의미한다고 보면 된다.  
  경로 탐색의 끝이며, 더 이상 탐색이 불가능하다.

- 단일 값 연관 필드  
  -> 단일 값, 즉 다른 하나의 엔티티를 대상으로 하는 연관관계 필드를 의미하며, @ManyToOne or @OneToOne 대상 엔티티이다.  
  묵시적 내부 조인이 발생하며, 해당 필드와 연관관계에 있는 연관 필드로 탐색이 가능하다.

- 컬렉션 값 연관 필드  
  -> 컬렉션 값, 즉 엔티티를 컬렉션으로 저장하고 있는 연관 필드를 의미하며, @OneToMonay or @MnayToMany 대상 엔티티이다.  
  묵시적 내부 조인이 발생하며, 더이상 탐색이 불가능하다. (size 메서드만 호출 가능)

### **_명시적 조인 VS 묵시적 조인_**

```java
select t from Member m join m.team t //명시적 조인
와 아래는 동일
select m.team from Member m //묵시적 조인

=> SQL 쿼리
select t.id, t.name
from MEMBER m inner join TEAM t
on m.team_id = t.id
```

m.team은 단일 값 연관 필드를 표현했으며, 쿼리가 나갈 때 해당 Member 엔티티와 연관된 "Team 엔티티"를 가져오기 위해서 SQL에서는 내부적으로 inner join이 발생한다.

실무에서는 명시적 조인으로 join이 발생하는 JPQL인 경우에는 쿼리로 풀어서 작성해야 한다.  
협업의 관점에서도, 유지보수 측면에서도 판단하기가 수월하기 때문이다.

</br>

---

### **_fetch join_**

페치 조인(fetch join)은 SQL 문법에는 존재하지 않는 조인이며, JPQL에서만 존재한다.

1. 성능 최적화를 위해 제공하는 기능
2. **_"연관된엔티티나 컬렉션을 SQL 한방쿼리로 함꼐 조회하는 기능"_** 이다.
3. inner join fetch, left outer join fetch 등 SQL join문법 뒤에 fetch를 붙이면 된다.

</br>

### **_기본 사용법_**

```java
JPQL
select m from Member m join fetch m.team t

SQL
select m.*(member 데이터 전부), T.*(해당 member와 연관 있는 team 데이터)
from Member m inner join Team t
on m.team_id = t.id
```

fetch는 위와 같이 해당 엔티티 뿐만 아니라 해당 엔티티와 연관되어 있는 엔티티의 정보들까지 한번의 쿼리로 가져오게 된다.

**_어떤 엔티티를 중심으로 연관되어 있는 엔티티를 가져올 것이냐는_**  
em.createQuery(query, Member.class) 에서 두번째 매개변수값으로 정해지니 잊지말자.

</br>

### **_지연 로딩에서의 N+1문제_**

즉시 로딩, 지연 로딩에서 즉시 로딩은 N+1문제(member만 조회하고 나서 즉시 로딩인 것을 보고, 연관관계인 Team에 대한 쿼리가 따로 추가적으로 발생)를 발생시키니 지연 로딩을 베이스로 깔고 가라 하였다.

하지만 알아두어야 할 것은 지연 로딩으로 바꾼다고 하여도 N+1문제는 발생할 수 있다.

```java
Team team = new Team();
team.setName("teamA");
em.persist(team);

Team team2 = new Team();
team2.setName("teamB");
em.persist(team2);

Member member = new Member();
member.setUsername("member");
member.setAge(10);
member.changeTeam(team);
em.persist(member);

Member member2= new Member();
member2.setUsername("member2");
member2.setAge(20);
member2.changeTeam(team2);
em.persist(member2);

em.flush();
em.clear();
```

위와 같이 member -> teamA에 member2 -> teamB에 속하는 상황이다.

```java
String query = "select m from Member m";

List<Member> result = em.createQuery(query, Member.class).getResultList();
```

이후 Member 테이블에 존재하는 레코드들을 대상으로 엔티티를 만든다. 그렇기에 member와 member2에 대한 엔티티 2개가 만들어져서 1차 캐시로 들어가 있을 것이며, member.team와 member2.team은 프록시 객체가 들어가 있는 상태이다.

지연로딩이기 때문에 member에 team에 접근할 때 프록시가 영속성 컨텍스트에 요청하여 실제 DB에서 연관되어 있는 team에 대한 쿼리를 날린다. 즉, 사용하고 싶을 때 그때 쿼리를 날리는 것이다.

해당 상태에서 아래의 로직을 돌린다면?

```java
for (Member member1 : result) {
System.out.println("member1 = " + member1);
/**
*   지연로딩에 의해 teamA에 대한 select쿼리 발생
*   만약 member2가 teamB에 속해있다면 teamB에대한 select쿼리 발생
*   100명이 member가 전부 다른팀이라면? 반복문이 실행될 때마다 Team에 대한 select쿼리가 발생
*   지연로딩이라고 해서 N+1문제가 해결되는 것은 아님
*   이런 경우라면 처음부터 join fetch로 연관된 데이터를 한방쿼리로 가져오는 것이 효율적
*/
System.out.println("member1.team = " + member1.getTeam());
}
```

for문에서 결과를 대상으로 하나씩 꺼내서 루프를 돌린다.  
그러면 쿼리는 어떻게 발생할까?

```java
Hibernate: "member엔티티를 대상으로 조회(result에 저장)"
    /* select
        m
    from
        Member m */ select
            member0_.id as id1_0_,
            member0_.age as age2_0_,
            member0_.TEAM_ID as TEAM_ID5_0_,
            member0_.type as type3_0_,
            member0_.username as username4_0_
        from
            Member member0_
member1 = Member{id=3, username='member', age=10} "1번째 for문에 member1 출력"

Hibernate:
    select
        team0_.id as id1_3_0_,
        team0_.name as name2_3_0_
    from
        Team team0_
    where
        team0_.id=?
member1.team = jpql.Team@5afbd567 "1번째 for문에 team을 출력하고자 할 때 프록시의 요청에 의해 쿼리 발생"
member1 = Member{id=4, username='member2', age=20} "2번째 for문에 member1 출력"

Hibernate:
    select
        team0_.id as id1_3_0_,
        team0_.name as name2_3_0_
    from
        Team team0_
    where
        team0_.id=?
member1.team = jpql.Team@608b1fd2 "2번째 for문에 team을 출력하고자 할 때 프록시의 요청에 의해 쿼리 발생"
```

member 엔티티를 대상으로 team에 접근할 때마다 쿼리가 발생하고 있다. 그 이유는 member와 member2가 다른 팀이기 때문이다. 만약 같은 팀이라면 처음 한번만 조회하고 이후에 접근할 때는 1차 캐시에 해당 팀에 대한 정보가 존재하니 쿼리를 발생시키지 않는다.

```
같은 팀이라는 것을 어떻게 알아서 쿼리를 안내보내고 1차 캐시에서 가져오는가? (위 예제에서는 프록시 객체에 target을 동일한 team으로 연결)

1. teamA에 대한 query 발생(teamA에 PK값을 알 수 있음)
2. 이후에 다른 member의 팀을 조회하는데 동일한 팀인 경우에는 query를 날리기 전에 where문에 member.team_id = team.id(teamA에 대한 PK값)
3. teamA에 대한 PK로 조회한 결과가 이미 존재하므로 쿼리 발생 X

간단히 말하면 같은 팀인 경우 같은 PK값으로 조건문에 사용한다. 근데 이전에 이미 조회한 결과로 해당 PK값을 가지는 엔티티가 존재하기 때문에 그냥 해당 엔티티를 바로 가져오게 되는것이다.
```

내용이 길어졌지만, 결국 다른 팀일때마다 계속 쿼리가 발생한다는 것이다.

이럴 경우에는 아래와 같이 fetch join으로 한번에 미리 가져와 놓고 사용하면 이후에는 쿼리가 발생하지 않는다.

```java
String query = "select m from Member m join fetch m.team t";
```

</br>

### **_중복 문제_**

**_하이버네이트 6.0이후에는 distinct 없이도 기본적으로 중복이 제거가되게 작동한다._**  
**_과거에는 이랬다 정도로만 알 것_**

```java
Team team = new Team();
team.setName("teamA");
em.persist(team);

Team team2 = new Team();
team2.setName("teamB");
em.persist(team2);

Member member = new Member();
member.setUsername("member");
member.setAge(10);
member.changeTeam(team);
em.persist(member);

Member member2= new Member();
member2.setUsername("member2");
member2.setAge(20);
member2.changeTeam(team2);
em.persist(member2);

Member member3 = new Member();
member3.setUsername("member3");
member3.setAge(10);
member3.changeTeam(team);
em.persist(member3);
```

위와 같이 teamA에는 meber와 member3가, teamB에는 member2가 들어가 있는 상태이다.

```java
String query = "select t from Team t join fetch t.members";
List<Team> result = em.createQuery(query, Team.class).getResultList();

//몇번 반복?
for (Team team1 : result) {
    System.out.println("member1 = " + team1 + " | team1.getMembers().size()" + team1.getMembers().size());
    for(Member m : team1.getMembers()){
        System.out.println("m = " + m);
    }
}
```

위와 같이 팀에 대한 엔티티 정보들과 해당 팀과 연관되어 있는 멤버들을 한꺼번에 조회하는 페치 조인 쿼리이다.

그러면 예상되는 결과로는 teamA에 대한 엔티티에 members 컬렉션에는 member와 member3가 들어있을 것이고, teamB에 대한 members에는 member2가 들어있을 것이다.

반복문은 팀 2개만 존재하니, 2번을 반복할 것이다.

</br>

반복문 결과

```java
member1 = jpql.Team@727320fa | team1.getMembers().size()2
m = Member{id=3, username='member', age=10}
m = Member{id=5, username='member3', age=10}
member1 = jpql.Team@17410c07 | team1.getMembers().size()1
m = Member{id=4, username='member2', age=20}
member1 = jpql.Team@727320fa | team1.getMembers().size()2
m = Member{id=3, username='member', age=10}
m = Member{id=5, username='member3', age=10}
```

하지만 반복문은 총 3번 반복되며, 2개는 같은 결과인 teamA에 대한 members가 1개만 다른 결과인 teamB에 대한 members가 나타난다.

그 이유는 SQL방식에서의 레코드 반환방식과 ORM의 차이인데, 위 한방 쿼리는 결국

```sql
select * from Team t inner join Member m on t.id = m.team_id;
```

이며, 3개의 레코드가 반환될 것이다.  
이를 객체지향 개념으로만 생각한다면 위와 같이 2개의 team 엔티티에 각각 2개, 1개 씩 들어가겠지만 JPA에서는 SQL의 입장과 맞출려다 보니 결국 내부적으로도 result에 동일한 엔티티를 2개, 다른 엔티티 1개 이렇게 3개를 담아버리게 되는 것이다.

그래서 이를 해결하기 위해서 distinct를 사용하면 되지만, 하이버네이트 6.0부터는 그냥 2개를 반환하는 것으로 변경되었다고 한다.

</br>

### **_fetch join 대상에는 별칭 불가_**

fetch join 대상에는 별칭을 줄 수 없다. t.members m <- 불가능 즉, 해당 별칭을 통해 조건으로 거르고 싶어도 불가능

**_그냥 연관된 데이터를 전부 가져올 뿐이므로, "조인패치 대상"으로 조건이 필요하면 패치 조인 사용 X_**  
 JPA는 연관관계로 객체 그래프 탐색을 했을 때 전부 있다고 가정하에 설계한 것

예제를 보면 바로 이해가 될 것이다.

```sql
select t from Team t join fetch t.members m where m.age > 10  -> X
select t from Team Member m inner join m.team t where m.age > 10  -> 0
```

조건으로 거르고 싶은 경우에는 fetch join이 아닌 풀어서 join해서 해결하자.

활용편2 내용과 아래 질문내용을 보고 추가한 내용  
https://www.inflearn.com/questions/15876/fetch-join-%EC%8B%9C-%EB%B3%84%EC%B9%AD%EA%B4%80%EB%A0%A8-%EC%A7%88%EB%AC%B8%EC%9E%85%EB%8B%88%EB%8B%A4

```
fetch join에서 별칭은 JPA에서 막았지만, 하이버네이트에서는 지원을 해준다. 하지만 "일관성"이 깨지는지 아닌지를 잘 판별하여 사용해야 한다.

"일관성"이란, 결국 fetch join은 연관된 데이터를 "전부" 가져오는데에 초점을 맞춘 JPQL에서만 존재하는 문법이다.
그런데 fetch join의 대상이 되는 엔티티에 별칭을 주어 on 또는 where절로 조건에 맞는 데이터만 가져오는 행위는 fetch join의 의도를 무시하는 행위이다. 그리고 이러한 행위가 바로 "일관성"을 의미한다.

정리하면, 별칭은 하이버네이트에서 사용이 가능하지만 사용하는 경우에는 일관성이 깨지지 않게 주의해서 사용하라는 것이다.
```

</br>

---

## **_엔티티 직접 사용_**

JPQL에서 엔티티를 직접 명시해서 사용하는 것은 SQL에서 해당 엔티티의 기본 키 값으로 변환해서 쿼리가 나간다.

```java
select count(m.id) from Member m //엔티티의 아이디를 사용
select count(m) from Member m //엔티티를 직접 사용

SQL 쿼리
select count(m.id) as cnt from Member m
```

아래와 같이 파라미터로 활용해서 사용이 가능하다.

```java
String jpql = “select m from Member m where m = :member”;
List resultList = em.createQuery(jpql)
.setParameter("member", member)
.getResultList();
```

위의 JPQL 쿼리를 아래의 JPQL쿼리로 변환할 수 있다.

```java
String jpql = “select m from Member m where m.id = :memberId”;
List resultList = em.createQuery(jpql)
.setParameter("memberId", memberId)
.getResultList();
```

간단히 말해서 그냥 id를 넣어서 생각하면 된다.

</br>

---

## **_NamedQuery_**

엔티티 클래스에 @NamedQuery 애노테이션으로 쿼리를 이름으로 정해서 해당 이름만 em.createNamedQuery에서(이름, 타입)과 같이 사용할 수 있다.  
또는 XML로 작성해놓고 사용이 가능하다.

해당 방법은 이후 spring data JPA에서 @Query로 추상화 되서 사용한다고 한다.

해당 방법은 매우 큰 이점을 가져온다.

애플리케이션 로딩 시점에 초기화 후 사용된다는 것이다. 이를 통해서 애플리케이션 로딩 시점에 해당 쿼리를 검증한다는 것이다.  
-> JPQL에서는 String타입으로 쿼리를 작성하다보니 직접 실행해야 검증이 되지만 해당 방법을 사용 시에 로딩 이후 쿼리 검증을 통해 바로 에러를 반환한다.

다음으로, 캐싱을 한다는 것이다. 그렇기에 메모리에 이점이 존재한다. (자주 사용하는 쿼리일수록 효율 증가)

</br>

---

## **_벌크 연산_**

그냥 대량의 update 또는 delete문을 작성할 때 사용하는 것이다.  
하나씩 루프로 돌리면서 하면 더티 체킹하고.. 쿼리나가고.. 채킹하고.. 쿼리나가고.. 등 비효율적인 작업으로 이루어지기 때문에 여러 테이블을 변경하는 경우에는 이 방법을 사용하자.

```java
int resultCount = em.createQuery(qlString)
.setParameter("stockAmount", 10)
.executeUpdate();
```

와 같이 맨 끝에 executerupdate를 호출해 주면 된다.

주의해야할 점은 벌크 연산은 영속성 컨텍스트를 무시하고 바로 DB에 쿼리를 날리기 때문에 만약에 벌크 연산 하고나서 find를 할 때 1차 캐시에 있는 정보를 가져오면 벌크 연산 이전의 결과를 사용하는 것이기 때문에 벌크 연산을 사용하는 경우 **_사용 후에 영속성 컨텍스트를 초기화를 해주자._**
