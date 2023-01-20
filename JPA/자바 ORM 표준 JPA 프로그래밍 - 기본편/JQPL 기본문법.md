# **_JPQL 기본 문법_**

해당 내용은 자바 ORM 표준 JPA 프로그래밍 - 기본편(김영한님)의 강의를 보고 정리한 내용입니다.

---

## **_객체지향 쿼리 언어_**

JPQL 공부하기 전에는 그저 JPA에서 제공하는 메서드만을 통해 간단한 select, delete, insert, update가 영속성 컨텍스트의 개념과 함께 알아보았다.

하지만 실제로 프로젝트를 진행하다보면 정말 복잡한 쿼리들이 등장하게 된다. 그럴경우 지금까지 학습했던 메서드만으로는 해결이 안되며, 이 때 사용하는 것이 JQPL 즉 "객체지향 쿼리 언어" 이다.

JPQL은 SQL을 추상화해서 객체지향적으로 작성하는 쿼리 언어라고 보면 된다.

```java
String jpql = "select m From Member m where m.name like ‘%hello%'";
List<Member> result = em.createQuery(jpql, Member.class)
.getResultList();
```

위는 JPQL을 통해 name에 hello가 들어간 경우에 Member와 관련된 데이터를 전부 가져오는 예제이다.

위에서 보면 알듯이 SQL이 테이블을 대상으로 쿼리가 진행된다면, 객체지향 쿼리 언어는 엔티티를 대상으로 쿼리가 진행된다.  
오해를 하면 안되는 내용이 있는데, JPQL로 작성한 쿼리는 결국 JDBC에 의해서 내부적으로 SQL언어로 변환되어 DB에 날리게 된다.

### **_Criteria와 QueryDSL_**

위의 JPQL은 결국 쿼리를 문자열로 작성하게 된다. 즉, 쿼리에 잘못된 문법이 적용되어도 문자열로 판단하고 컴파일 시점에서는 에러를 찾을 수 없다는 것이다.

다음으로, 동적 쿼리를 작성하기가 매우 까다롭다. 문자열이기 때문에 더해가는 식으로 쿼리를 붙여나가야 하기 때문이다.

이러한 문제점들을 해결하는 수단이 Criteria와 QueryDSL이다.  
하지만 Criteria는 너무 복잡하고, 직관적이지가 않기 때문에 유지보수에도 매우 힘들다. 그래서 실무에서는 QueryDSL을 사용한다.

```java
JPAFactoryQuery query = new JPAQueryFactory(em);
QMember m = QMember.member;
List<Member> list =
query.selectFrom(m)
.where(m.age.gt(18))
.orderBy(m.name.desc())
.fetch();
```

위가 QueryDSL 코드이다. 메서드체이닝으로 query가 어떻게 만들어지는지 직관적이다.

```
QueryDSL도 결국은 JPQL을 좀 더 쉽게 활용할 수 있는 수단일 뿐이다.
결국 정말로 복잡한 쿼리같은 경우나 SQL문으로 해결할 수 밖에 없는 쿼리들은 네이티브 SQL이나 SQL Mapper 기술(JDBC Template, MyBatis)를 사용해야 한다는 것이다.
```

</br>

---

## **_프로젝션(SELECT)_**

```java
Member member1 = new Member();
member1.setUsername("김철수");
member1.setAge(10);
em.persist(member1);

//createQuery로 부터 나온 Member들을 List에 담는데, 그럼 해당 Member들은 영속성 컨텍스트에 관리가 될까?
List<Member> result = em.createQuery("select m from Member m", Member.class).getResultList();
Member findMember = result.get(0);
findMember.setAge(20); //update문 발생, 즉 결과로 나온 모든 Member 엔티티 객체들은 전부 영속성 컨텍스트에 관리 됨

```

주석으로 적어놨지만, 다시 한번 정리하자면 persist 시점에는 영속성 컨텍스트의 1차 캐시에만 존재하지 flush를 진행하지 않았다. 즉, DB에는 아직 쿼리가 날라가기 전이라는 것이다.

이 상태에서 JPQL로 조회를 해오면 어떻게 되는가?  
정답은 JPQL 쿼리를 날리게 될 땐 flush()가 자동으로 동작하고나서, 쿼리가 실행이 된다.

다음으로, 위에는 member1을 넣고 조회하는 것이지만 만약 이미 데이터베이스에 다른 레코드들이 이미 존재한 상태이다.(member1까지 포함하여 10개가 있다고 가정)

그러면 result에는 당연히 10개의 멤버들이 담길것이다.  
여기서 의문인 점은 em.find를 해서 1차 캐시에 없어서 DB에 접근해서 가져오는 경우에는 리플렉션 기술을 통해서 엔티티 객체를 만들어 1차 캐시에 넣어둔다는 것이다.

그럼 위와 같이 JPQL로 여러개의 멤버들을 가져오게 되는 경우에는 어떻게 동작하는가?  
바로 10개를 가져왔다면 10개 모두 엔티티로 만들어 1차 캐시에 넣어둔다는 것이다.  
마지막 코드에 List에 첫번째 Member를 가져와서 set만 했을 뿐인데 update 쿼리가 나가게 된다. 즉, 1차 캐시에 존재하여 더티 체킹이 발생했다는 것을 알 수 있다.

### **_N+1문제는 항상 의식해라_**

```java
List<Member> teamList = em.createQuery("select m from Member m", Member.class).getResultList();
```

즉시로딩, 지연로딩 파트 마지막에서도 설명했지만 위와 같이 즉시로딩인 상태에서 쿼리를 날리게 되면 Member에 대한 select쿼리를 날린 후에 team이라는 필드의 즉시로딩을 보고서 다시 한번 Team 테이블을 대상으로 select 쿼리가 나가게 된다.(team 엔티티가 1차 캐시에 존재한다면 Team 테이블에 대한 쿼리는 안나감)

그러니 지연로딩을 무조건 기본으로 세팅하자.

</br>

### **_조인 쿼리는 적어줄 것_**

```java
//join query 발생
List<Team> teamList = em.createQuery("select m.team from Member m", Team.class).getResultList();
//위와 동일 쿼리 발생(아래의 쿼리를 통해 직관적으로 보기 좋음 그래서 join은 명시적으로 적어줄 것)
List<Team> teamList2 = em.createQuery("select t from Member m join m.team t", Team.class).getResultList();
```

위 두 JPQL쿼리는 동일한 쿼리가 발생한다. 하지만 아래 쿼리가 join을 한다는 것을 직관적으로 알 수 있어서 협업할 때나 코드를 다시 리팩토링할 때 등 여러 이점이 존재한다.

### **_임베디드 타입 조회는 어떻게?_**

```java
//임베디드 타입 프로젝션
            List<Address> addressList = em.createQuery("select o.address from Order o", Address.class).getResultList();
```

Order 엔티티에는 Address가 임베디드 타입으로 선언되어 있는데, 이럴 경우 JPQL은 위와 같이 쿼리를 하면 된다.

### **_스칼라 값 타입_**

```java
//스칼라 타입 프로젝션(1번째 방법)
//Object 타입으로 저장함
List scalaTypeList = em.createQuery("select m.useranme, m.age from Member m").getResultList();

Object o = scalaTypeList.get(0);
Object[] objectArr = (Object[]) o;
System.out.println("useranme = " + objectArr[0]);
System.out.println("age = " + objectArr[1]);

//2번째 방법
//Object배열로만 들어갈 수 있도록 처음부터 제한(형변환 필요없어짐)
List<Object[]> resultList = em.createQuery("select m.useranme, m.age from Member m").getResultList();

//3번째 방법(제일 깔끔함, memberDTO의 패키지가 길어지면 쿼리도 같이 길어짐)
List<MemberDTO> dtoList = em.createQuery("select new jpql.MemberDTO(m.username, m.age) from Member m", MemberDTO.class).getResultList();
```

SQL에서는 조회하고 싶은 컬럼들만을 select 절에서 적음으로써, 가져올 수 있다. 이를 JPQL에서는 어떻게 해야하는가?

위처럼 3가지 방법이 존재한다.

여러 타입들을 한번에 저장하기 위해서 List는 Object[]로 담는다.  
1번째 방법은 제네릭으로 사용하지 않았기 때문에 형변환을 한 번 거쳐야 하지만 2번째 방법은 제네릭에 의해서 바로 사용이 가능하다.

3번째 방법이 제일 깔끔한 방법으로써, 여러 타입들을 저장하고자 하는 DTO를 만들어서 new를 통해 엔티티가 아닌 DTO에 담는 방법이다.

### **_파라미터 매핑_**

```java
List<Member> result2 = em.createQuery("select m from Member m where username =:username ", Member.class)
.setParameter("username", "김철수").getResultList();
```

콜론(:)으로 파라미터명을 지정하고, 이후 setParameter로 해당 파라미터명과 값을 매핑하면 된다.

</br>

---

## **_서브쿼리_**

```java
select m from Member m
where m.age > (select avg(m2.age) from Member m2)
```

간단한 서브쿼리 문법이며, SQL문법과 크게 다를게 없다.  
단, JPA는 서브쿼리를 where절과 having절에만 가능하며, 하이버네이트에서는 select절까지 서브쿼리를 적용할 수 있도록 되어있다.

하지만 FROM절의 서브쿼리(인라인 뷰)는 불가능하다. 보통 인라인 뷰의 서브쿼리는 조인으로 해결할 수 있으며, 정말 필요하다면 네이티브 쿼리나 SQL Mapper기술을 활용하자.

</br>

---

# **_정리_**

이 외에도 페이징문법, Case식(조건식), SQL에서 사용가능한 함수들 중 대부분(?)을 사용할 수 있다. 방언에서 제공안해주는 함수들의 경우 사용자 정의 함수를 통해 추가할 수 있다.

하지만, 윈도우 함수는 잘 지원안해주는 것 같으므로, 추가를 해주거나 아니면 네이티브 쿼리를 사용하자.  
-> http://querydsl.com/static/querydsl/4.0.1/reference/ko-KR/html_single/#d0e1162  
 QueryDSL에서 SQLExpressions 클래스를 통해서 윈도우함수를 지원한다고 한다.

결국, JPQL은 SQL과의 문법적인 면에서 차이는 테이블을 대상으로 쿼리를 사용하냐, 엔티티를 대상으로 쿼리를 사용하냐인 것 같다.  
이 외의 다른 사용방식은 거의 유사한 것 같다.
