# **_OSIV_**

해당 내용은 실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화(김영한님)의 강의를 보고 정리한 내용입니다.

---

OSIV(Open-Session-In-View)는 "영속성 컨텍스트의 생존 범위를 어떻게 할지"를 정하는 속성이다.  
기본값은 true이며, true면 OSIV ON이며, false면 OSIV OFF이다.  
spring.jpa.open-in-view로 속성을 정의할 수 있다.

</br>

---

## **_OSIV ON(Default)_**

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/213917985-d9b2158d-06c8-4305-874e-c4b9b0842aef.png" width = 70%>
</p>

기본값인 true일 때의 영속성 컨텍스트 생존 범위를 나타낸다.  
해당 속성을 알기 전까지는 트랜잭션이 시작되었을 때 영속성 컨텍스트가 만들어지며, 트랜잭션이 커밋 또는 롤백되는 시점에 영속성 컨텍스트가 제거가 되는지 알았지만 자세히 보면 다르게 동작되는 것을 알 수 있다.

트랜잭션이 시작되었을 때 영속성 컨텍스트가 만들어지는 것은 동일하지만, 트랜잭션이 종료가 되어도 영속성 컨텍스트는 그대로 존재한다.(단, 읽기만 가능한 상태가 된다)

최종적으로 요청(Request)에 대한 응답(Response)이 될 때 영속성 컨텍스트가 제거된다.  
API방식이라면 데이터를 응답했을 때, SSR방식이라면 View에 의해 렌더링이 되었을 때 제거가 된다.(response객체를 반환할 때)

```java
@GetMapping("/api/v1/orders")
public List<Order> ordersV1() {
    List<Order> all = orderRepository.findAllByString(new OrderSearch());
    for (Order order : all) {
        order.getMember().getName();
        order.getDelivery().getAddress();
        List<OrderItem> orderItems = order.getOrderItems();
        orderItems.stream()
                .forEach(o -> o.getItem().getName());
    }
    return all;
}
```

v1코드이다.  
지연 로딩에 의해서 member와 delivery 객체는 프록시 객체로 할당되며, 프록시 객체를 반환해버리면 에러가 발생되기 떄문에(Hibernate5Module을 빈으로 등록해서 사용한다면 에러말고 null을 반환) 데이터를 정상적으로 반환하기 위해서 Controller단에서 for문을 돌려 getName과 getAddress를 강제로 호출하여 DB에 접근하여 프록시 객체를 실제 target과 연결하고, 이후 데이터를 가져온다.

이를 통해 정상적으로 데이터가 반환되는 것을 확인할 수 있다. 근데 여기서 의문점을 가졌어야 했다. 어떻게 트랜잭션 범위 밖인 Contorller단에서 DB에 접근할 수 있었던 걸까?  
바로 OSIV가 true로 설정되어 있었기 때문이다. 그래서 트랜잭션 범위 밖에서도 영속성 컨텍스트가 존재하며, 데이터베이스 커넥션 또한 가지고 있기 때문에 DB에 접근할 수 있었던 것이다.

</br>

---

## **_OSIV OFF_**

<p align = "center">
<img src="https://user-images.githubusercontent.com/62879192/213918435-a3ac98da-59d8-424f-85c6-e3a620e375ac.png" width = 70%>
</p>

속성 값을 false로 주면, 위와 같이 영속성 컨텍스트의 생존 범위는 트랜잭션이 시작될 때 생성되고, 종료될 때 제거가 된다.

### **_false로 설정하고 v1을 호출하면?_**

org.hibernate.LazyInitializationException: could not initialize proxy - no Session 과 같은 에러가 발생해버린다.

</br>

---

## **_OSIV 트레이드 오프_**

OSIV를 켜두면, v1 코드와 같이 Controller단에서도 언제든지 지연로딩을 수행할 수 있다. 즉, 영속성 컨텍스트의 생존 범위에 대해 크게 신경쓸 필요가 없다는 것이다.  
하지만 응답을 할 때까지 DB 커넥션을 계속 가지고 있어서 만약 외부 API호출과 같은 응답이 오래걸리는 로직이 존재하는 경우 그 시간만큼 커넥션을 계속 가지고 있는다는 것이다.

반대로 OSIV를 꺼두면, 영속성 컨텍스트의 생존 범위에 대해 신경쓰면서 로직을 작성해야 한다. 위 v1와 같이 Controller단에서 하던 행위들을 트랜잭션으로 몰아 넣어서 작성을 한다던가, 처음부터 지연 로딩이 아닌 패치 조인을 한다던가와 같이 여러 방법에 대해 고민을 해야한다.

김영한님은 보통 커맨드와 쿼리를 분리하는 CQRS패턴을 사용한다고 한다.  
CQRS는 간단히 말하면 Command 즉, Create, Update, Delete와 같은 갱신 쿼리들을 행하는 Service와 Repository를 하나씩 만들고, Query 즉, Select하는 부분에 대한 Service와 Repository를 또 따로 하나를 만드는 것이다. 그리고 쿼리에 대한 서비스는 트랜잭션을 readonly = true를 지정해주는 것이다.

위는 CQRS를 매우 간단하게 풀어서 설명한 것이고, 검색을 해보면 정말 다양한 방법으로 CQRS 패턴을 도입하는 것 같다.

### **_OSIV ON? OFF?_**

OSIV를 켜두면 커넥션을 오래 가지고 있기 때문에 커넥션이 마를 수가 있다. 그래서 김영한님은 보통 트래픽이 많은 고객 서비스의 경우에는 OSIV를 꺼두고, ADMIN과 같은 커넥션이 적게 사용하는 부분에서는 OSIV를 켜둔다고 한다.
