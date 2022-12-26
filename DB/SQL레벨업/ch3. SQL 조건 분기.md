# **_SQL 조건 분기_**

해당 내용은 [SQL 레벨업](http://www.yes24.com/Product/Goods/24089836) 책을 통해 공부한 내용입니다.

쿼리문에 의한 결과 테이블은 쿼리문만을 보고 예상하고, 예상이 안되는 건 다시 직접 실행

ch3은 MySql문법을 기준

---

</br>

ch2 에서 case식으로 조건 분기를 하는 예시를 봤다. 그런데 case식이 아닌 union 으로도 조건 분기에 사용할 수 있는데, 해당 챕터에서는 둘의 차이점과 무엇을 언제 사용해야 하는지를 살펴본다.

</br>

---

## **_UNION과 CASE 비교 예제_**

Items(상품) 테이블에 세전 가격을 표시하는 필드와 세후 가격을 표시하는 필드가 각각 존재하며, 상품의 제작연도가 2002년 이전이면 세전가격을 이후면 세후가격을 보여주도록 하는 예제의 경우 이를 UNION으로 작성해보자.

```sql
select item_name, year, price_tax_ex as price
from items
where year <= 2001
union
select item_name, year, price_tax_in as price
from items
where year >= 2002;
```

2002년 이전인 경우 아이템이름, 연도, 세전가격(pirce 별칭)을 보여주는 테이블과 2002년 이후인 경우 아이템이름, 연도. 세후가격(price 별칭)을 보여주는 테이블을 union을 통해 합해서 보여준다.

이제 이를 CASE문으로도 동일하게 작성해보자.

```sql
select item_name, year,
case when year < 2002 then price_tax_ex
else price_tax_in
end as price
from items;
```

위와같이 select 구문에 case식으로 조건에 따라 어떠한 식을 사용할지를 정하는데, 위에서는 세전가격을 사용할지 세후가격을 사용할지를 정한다.

union과 case식은 동일한 결과를 보여주는데 둘의 차이는 무엇일까?

### **_차이점_**

둘의 차이점은 실행계획에서 나타난다. (쿼리문 앞에 explain을 붙여 직접 확인해보자)

union에는 select 문이 2개가 들어간다. 즉, 테이블 조회가 2번이 발생하게 된다. 그에 비해 case에는 단 한번의 테이블 조회만이 발생한다.

예제용 테이블에는 데이터가 적게 들어가 있으니 성능에는 차이가 없겠지만, 만약 엄청난 수의 데이터가 존재하는 테이블을 조회하는 경우에는 차이가 심할 것이다.

그래서 union과 where 절로 조건 분기를 하는 것은 옳은 방법이 아니라고 한다.

</br>

---

## **_집계 함수에 case_**

성별에 따라 지역별 인구수를 보여주는 예제이다.

```sql
select prefecture,
sum(case when sex = 1 then pop else 0 end) as pop_man,
sum(case when sex = 2 then pop else 0 end) as pop_wom
from population
group by prefecture;
```

위와 같이 집계함수 내부에도 case식을 사용할 수 있는데, 이를 완벽히 이해하기 위해서는 절차지향적인 사고에서 벗어나서 "식"이라는 사고(관점)을 가지고 구문을 봐야한다.

case는 말 그대로 해당 조건에 따라 식을 반환하는 것이다. 그렇기에 sum 내부에 식이 반환되면 해당 식으로 집계를 하는 것이다.

참고로 위를 union으로 변경할 수 있다.

```sql
select prefecture, sum(pop_men) as pop_men, sum(pop_wom) as pop_wom
from (select prefecture, pop as pop_men, null as pop_wom
	  from population
      where sex = 1
      union
      select prefecture, null as pop_men, pop as pop_wom
      from population
      where sex = 2) tmp
group by prefecture;

```

from 뒤에 오는 것은 결국 어떠한 테이블을 기준으로 실행할 것인지 이다.  
결국 서브쿼리의 결과가 테이블로 보여지므로, from 절 뒤에 작성할 수 있다.

</br>

---

## **_집약 결과로 조건 분기_**

직원이 소속된 팀이 1개일 경우에는 팀 이름을 그대로 반환하고, 2개일 경우 "2개를 겸무", 3개 이상일 경우에는 "3개 이상을 겸무" 를 반환토록하는 예제이다.

```sql
select emp_name, team
from employees
group by emp_name
having count(*) = 1
union
select emp_name, '2개를 겸무' as team
from employees
group by emp_name
having count(*) = 2
union
select emp_name, '3개 이상을 겸무' as team
from employees
group by emp_name
having count(*) >= 3;
```

위는 union 방법으로 집약 결과를 조건 분기한 예제이다.

</br>

```sql
select emp_name,
case when count(*) = 1 then team
	 when count(*) = 2 then '2개를 겸무'
     when count(*) >= 3 then '3개 이상을 겸무'
end as team
from employees
group by emp_name;
```

위는 select 구문에 case식을 활용하여 집계 결과에 조건 분기를 한 것이다.

실행 계획을 안봐도, 무엇이 쿼리가 간결하고, 성능이 좋을지 예측이 된다.

where절로 조건 분기를 하는 것과 마찬가지로 having에 조건 분기를 하는 것 또한 옳지 않는 방법이니 주의를 하자.

</br>

---

## **_그럼 UNION은 언제?_**

union을 조건 분기로 사용하는 경우는 크게 2가지로 볼 수 있다.

1. 하나의 테이블이 아닌 여러 테이블에서 사용해야 하는 경우
2. 테이블에 인덱스가 적용되어 있는 경우

1번의 경우 case로도 할 수는 있지만 그러면 필연적으로 테이블끼리 "결합" 이 발생하여 성능 저하가 발생할 수 있다고 한다.

2번의 경우 뒤 10장 인덱스 내용에서 자세히 살펴볼 예정이며, 예제에서는 3회의 인덱스 스캔(union 사용)과 1회의 테이블 풀 스캔(on, in, case 여러 방법 소개)을 비교하며, **_테이블 크기와 검색 조건에 따른 레코드 히트율_** 에 따라 무엇이 성능이 좋을지 비교해야 하며, 테이블이 크고 where 조건으로 선택되는 레코드가 적을수록 union이 더 빠르다고 한다. (많은 레코드에서 명확한 where구문의 조건하에)

책에서 on, in, case 다방면으로 예시를 들어주며, case의 조건판별에 따른 문제점을 문제로 내주니 참고(직접 작성한 쿼리문은 다른 폴더에 정리해뒀으니 복습할 때 참고)
