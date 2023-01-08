# **_집약과 자르기_**

해당 내용은 [SQL 레벨업](http://www.yes24.com/Product/Goods/24089836) 책을 통해 공부한 내용입니다.

쿼리문에 의한 결과 테이블은 쿼리문만을 보고 예상하고, 예상이 안되는 건 다시 직접 실행

ch4는 MySql문법을 기준

---

해당 챕터에서는 집약과 자르기 내용으로 group by와 윈도우 함수를 쓰기 위한 over([partition by] [order by]) 의 내용을 다룬다.

---

## **_집약 함수_**

집약 함수에는 COUNT, SUM, MAX, MIN, AVG 함수 등이 존재하며, 이 외의 다른 집약 함수들도 존재하지만 위 5개가 표준SQL에서 제공하는 함수들이다.

해당 함수들을 활용한 예제가 책에서는 2개가 존재하지만 해당 내용에서는 한개를 예시로 들어보자.

해당 예시는 143pg에서 여러 제품들 중에서 사용 가능한 연령대가 0 ~ 100세인 제품만을 추출하는 예제이다.

```sql
select reserve_id,
sum(low_age),
sum(high_age)
from pricebyage
group by reserve_id
having sum(high_age - low_age + 1) = 101;
```

위 실행 계획을 보면 테이블을 풀 스캔 후 group by 연산을 해시 방법을 통해서 진행한다.

위 식을 보기 전에 아래의 예제로 group by의 문제점을 보자.

```sql
select reserve_id, low_age, high_age, max(price)
from pricebyage
group by reserve_id;
```

위 쿼리문을 실행시켜보자.  
첫 번째 레코드에 제품1, 0, 50, 3000이 찍히는데, 전체 조회를 해서 결과를 비교해보면 제품1, 0, 50에 가격은 2000이다. 그런데 위 쿼리문에서는 이상한 결과가 보이는데, 실상은 제품1을 가진 여러 레코드들 중에 첫번째 제품1의 reserve_id, low_age, high_age를 보여주고 가격만 여러 제품1들 중에서 최고가격을 보여주는 것이다.

그 이유는 price는 같은 그룹들의 여러 값들 중에 어떠한 값을 반환해야 할지 명확하지만 나머지 필드의 값들은 그룹 내에 어떠한 값을 사용할지 모르기 때문이며, sql_mode에 설정이 되어 있다면 위 경우에 에러를 반환하지만 보통 기본은 무작위로 추출해서 보여준다.

참고 : https://school.programmers.co.kr/questions/38703

다시 돌아와서 0 ~ 100세 연령대의 제품을 출력하는 쿼리문을 보자.  
having 절에 의해서 여러 그룹 중에 해당 조건에 만족하는 그룹만을 반환하도록 명시해 주었고(제품1 그룹, 제품2 그룹, 제품 3 그룹 등등 여러 그룹 중 해당 조건을 만족하는 그룹만을 사용 위에서는 제품1 그룹과 제품 4그룹만을 반환), 이후 reserve_id와 sum함수에 의한 나이값을 보여준다.

근데 해당 쿼리문의 select 절에 low_age, high_age 을 추가한다면 동일하게 어떠한 값을 사용해야 할지 모르기 때문에 보통 첫번째 레코드의 값을 사용한다.  
그러니 주의하도록 하자.

</br>

---

## **_자르기_**

group by는 "집약" 기능과 "자르기" 기능, 두 기능을 사용하는 구문이며, 위 집약 함수에서는 "집약" 기능을 강조해서 보여주었고 해당 내용에서는 "자르기" 기능을 강조해서 설명한다.

20세 미만은 "어린이", 20~ 69세는 "성인", 70세 이상은 "노인" 이라는 집합 구조로 자른 집합들에 명칭을 보여하고, 해당 집합에 포함되는 수를 출력해 보자.

```sql
select case
when age < 20 then '어린이'
when age >= 20 and age < 70 then '성인'
when age >= 70 then '노인'else null end as age_class, count(*)
from persons
group by case
when age < 20 then '어린이'
when age >= 20 and age < 70 then '성인'
when age >= 70 then '노인'else null end;
```

from -> group by -> select 순으로 실행이 될 것이며, **_group by절에 필드만 들어갈 수 있다고 생각하는 경우가 많은데, "식"이 들어갈수도 있다는 걸 반드시 기억하자._**

group by 절에서 20세 미만은 "어린이" 그룹에, 20 ~ 69세는 "성인" 그룹에, 70세 이상은 "노인" 그룹으로 나누었다.  
필드로 사용하는 경우에는 같은 값으로만 그룹을 나누었지만, 위 식을 사용하면 조건의 개념으로 그룹을 나눌 수 있다.

그리고 select절에서는 group by에 문제때문에 age를 어떠한 값을 사용할지 몰라서 보통 해당 그룹에 첫번째 값을 사용할 것이다. 하지만 조건을 기준으로 나누었기 때문에 해당 그룹의 어떠한 값을 사용해도 동일한 조건만을 통과할 것이기 때문에 위와 같은 방법이 사용이 가능하다. (group by의 식과 다른 식을 사용하게 되면 or 조건이 이상하면 당연히 원하는 결과가 안나올 것)

그리고 count 집약 함수를 통해 해당 그룹의 수를 반환한다.

</br>

---

## **_partition by (윈도우 함수)_**

마지막으로 윈도우 함수를 사용하기 위한 **_윈도우 함수 over([partiontion by], [order by])_** 를 알아보자.

해당 구문은 group by의 "자르기" 기능만을 사용한다고 봐도 무방하다.  
즉 group by는 "집약"을 통해 그룹간의 통계를 내는데 보통 사용하지만 해당 구문은 "집약"을 하지 않기 때문에 출력되는 레코드 수 또한 "집약" 되지 않고 그대로 출력한다.

154페이지에 해당 구문을 사용하지만, 이번엔 프로그래머스에서 가져온 문제를 통해 알아보자.

출처: https://school.programmers.co.kr/learn/courses/30/lessons/131123

</br>

```sql
select food_type, rest_id, rest_name, favorites
from
(
select food_type, rest_id, rest_name, favorites, rank() over(partition by food_type order by favorites desc) as r
from rest_info
) as tmp
where r=1
order by food_type desc;
```

우선 위 식은 group by를 안쓰고 윈도우함수만을 통해서 "자르기" 작업을 한 후에 랭킹이 1등인 레코드만 추출해서 보여준다.

아래는 group by를 활용한 다른 정답이다.

```sql
select food_type, rest_id, rest_name, favorites
from (select food_type, rest_id, rest_name, favorites, rank() over(partition by food_type order by favorites desc) as rnk from rest_info) as tmp
group by food_type
(having min(rnk) or having min(favorites)) 붙여도 정답
order by food_type desc;
```

윈도우 함수에 partition by food_type으로 자를 기준을 음식 종류로 지정하고, 같은 그룹 내에서 랭킹을 맺도록 하였다.

의문인게 select에 food_type, rest_id, rest_name, favorites는 명확히 어떠한 값을 써야하는지 모른다. 그런데 정답 처리가 된다는 건데 이해가 안되서, 154페이지의 예제를 살짝 바꾸어서

```sql
select name, age, age_class, age_rank_in_class
from(
select name, age, case
when age < 20 then '어린이'
when age >= 20 and age < 70 then '성인'
when age >= 70 then '노인'else null end as age_class,
rank () over(partition by case
when age < 20 then '어린이'
when age >= 20 and age < 70 then '성인'
when age >= 70 then '노인'else null end order by age desc) as age_rank_in_class
from persons
) as tmp
group by age_class;
```

를 실행시켜봤을 때, select 구문에 어떠한 명확한 조건도 없는데도 무작위가 아닌 랭킹1등에 만족하는 모든 값들을 그대로 보여준다.

참고 : http://jason-heo.github.io/mysql/2014/03/05/char13-mysql-group-by-usage.html

를 참고해보면 group by에 명시하지 않은 필드 또는 집약 함수를 통한 필드 접근 외에는 잘못된 sql 문법이다. (mysql만 이상하며, sql_mode로 제어 가능)

그럼 위 방법말고 어떻게 접근해야 할까?

출처: https://school.programmers.co.kr/questions/40104

```sql
SELECT FOOD_TYPE , REST_ID , REST_NAME , FAVORITES
FROM REST_INFO
WHERE (FOOD_TYPE , FAVORITES) IN
      (SELECT FOOD_TYPE , MAX(FAVORITES) from REST_INFO GROUP BY FOOD_TYPE)
ORDER BY FOOD_TYPE DESC
```

위 출처에서 다른 분이 oracle 문법으로 작성한 쿼리문이다.

where절에 서브쿼리로 들어간 부분을 보자.

group by에 food_type이 들어가고 select구문에 food_type와 집약 함수 MAX 만을 사용한다. 그리고 이를 in절로 판별한다.

</br>

---

## **_무조건 알아두자_**

해당 챕터 이후 group by를 사용하는 경우 select 구문에는 반드시 group by에 명시한 필드 또는 집계 함수 외에는 사용하지 말자.
