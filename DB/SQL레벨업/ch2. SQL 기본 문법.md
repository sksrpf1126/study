# **_SQL 기본 문법_**

해당 내용은 [SQL 레벨업](http://www.yes24.com/Product/Goods/24089836) 책을 통해 공부한 내용입니다.

정말 간단한 기본 쿼리문은 제외

쿼리문에 의한 결과 테이블은 쿼리문만을 보고 예상하고, 예상이 안되는 건 다시 직접 실행

ch2는 MySql문법을 기준

---

## **_SELECT 구문_**

</br>

### **_뷰와 서브쿼리_**

1. 뷰는 사용자에게 접근이 허용된 자료만을 제한적으로 보여주기 위해 하나 이상의 기본 테이블로부터 유도된, 이름을 가지는 가상 테이블이다.

2. 뷰는 저장장치 내에 물리적으로 존재하지 않지만 사용자에게 있는 것처럼 간주된다.

3. 뷰는 조인문의 사용 최소화로 사용상의 편의성을 최대화 한다.

라고 "뷰"를 정의하는데, 해당 책에서는 뷰를 **_"SELECT 구문"_** 을 저장하는 것 뿐이라고 한다.

쉽게 이해하기 위해 예시를 통해 보자.

```sql
create view CountAddress (v_address, cnt)
AS
select address, count(*)
from address
group by address;
```

```sql
select *
from CountAddress;
```

뷰를 만들고 위와 같이 조회를 해보면, 주소로 그룹을 나누고, 그룹의 수와 주소를 필드로 가지는 가상의 테이블이 만들어진다.

이후 select로 전체 조회를 하면 당연히 2개의 필드로 이루어진 레코드들이 나타날 것이다.

근데 뷰는 select 구문을 저장하는 것 뿐이라고 한다면, 뷰를 만들지 않고 select 구문으로 한번에 사용할 수 잇지 않을까?

서브 쿼리를 사용하면 해결이 된다.

```sql
select v_address, cnt
from (select address as v_address, count(*) as cnt
      from Address
      group by address) as CountAddress;
```

위 쿼리를 사용하면 뷰의 결과와 같은 결과를 반환한다.  
그렇기에 뷰를 "select 구문만을 저장하는 것 뿐" 이라고 해당 책에서 정의한 것이다.

</br>

---

## **_SQL 조건 분기_**

c, java, javascript 등의 프로그래밍 언어의 경우에 대부분 if ~ else 문과 switch 문으로 조건식을 사용한다.

SQL문법은 약간 다르게

```sql
CASE WHEN [조건식(평가식)] THEN [식]
     WHEN [조건식(평가식)] THEN [식]
     WHEN [조건식(평가식)] THEN [식]
     ELSE [식]
END
```

의 문법으로 사용하게 된다.

예시를 보자.

```sql
select name, address,
case when address = '서울시' then '경기'
when address = '인천시' then '경기'
when address = '부산시' then '영남'
when address = '속초시' then '관동'
when address = '서귀포시' then '호남'
else null end as district
from address;
```

위와같이 조건문의 경우에는 select, where, group by, having 등 다양한 곳에서 사용할 수 있다.

위는 district 필드에 '서울시' 주소를 가진 사람의 경우에는 '경기'로 데이터가 들어가며 아래의 조건들 또한 조건식에 따라 해당 데이터가 들어간다.
