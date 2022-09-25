# **_IOC와 DI_**

spring(boot)을 검색하면 가장 많이 나오는 단어가 IOC(제어의 역전)와 DI(의존성 주입)이다. 그만큼 스프링의 핵심 키워드라고 말할 수 있다.  
IOC는 기존에 객체들을 개발자가 직접 생성하고 호출하며 제어를 했다면, Spring 프레임워크에서는 개발자가 아닌 외부 즉, 스프링 측에서 해당 객체들을 전부 제어한다는 것이다. 그래서 제어가 역전(개발자에서 프레임워크로)되었다고 하여 "제어의 역전"이라고 부르는 것이다.  
spring(boot)를 써볼 때 우리는 컨트롤러, 서비스, 레포지토리 등 해당 클래스와 메서드들만 스프링에 맞게끔 구현만 하고, 해당 객체들이 어느 시점에서 호출될 지는 신경쓰지 않는다. 그저 스프링이 알아서 필요한 시점에 가져다가 쓸 뿐이다. (해당 객체들은 IOC 컨테이너 또는 DI 컨테이너라 불리는 곳에서 빈이라는 이름으로 관리가 된다.)

다음으로 의존성이란 A가 B를 의존한다. 쉽게 표현하자면 B가 변화하면 B를 의존하고 있는 A에도 영향을 미친다. 라고 말할 수 있다.  
그럼 의존성을 **_주입_** 한다는 말은 무슨 말일까?  
주입이라는 단어에서 연상되는 것은 백신 주사(외부)로부터 내 몸(내부)으로 백신이 주입되는 그런 이미지가 연상이 된다.  
그럼 의존성 주입은 외부로부터 의존성을 주입받는다 라고 표현이 가능하다.

</br>

---

## **_DI(의존성 주입)_**

예시를 보면서 DI에 대해 이해를 해보자.

```java
interface Order {
    void adeOrder();
}

class AdeMaker implements Order {

    private AdeRecipe ade = new StrawBerryAde();

    @Override
    public void adeOrder() {
        ade.adeMake();
        System.out.println("고객에게 음료를 전달합니다.");
    }
}


interface AdeRecipe {
    void adeMake();
}

class StrawBerryAde implements AdeRecipe {

    private final int SUGAR = 20; //20g
    private final int STRAWBERRY = 5; //5개
    private final int SPARKLING_WATER = 300; //300ml

    @Override
    public void adeMake() {
        System.out.println(this.toString() + " 를 준비합니다.");
        System.out.println("딸기를 씻고, 꼭지는 떼어냅니다.");
        System.out.println("믹서기에 재료를 넣어 갈아줍니다.");
        System.out.println("얼음과 함께 컵에 담습니다.");
    }

    public String toString(){
        return "설탕 : " + SUGAR + "g " + "딸기 : " + STRAWBERRY + "개 " + "탄산수 : " + SPARKLING_WATER + "ml";
    }
}

class GrapeAde implements AdeRecipe {

    private final int SUGAR = 30; //20g
    private final int Grape = 7; //5개
    private final int SPARKLING_WATER = 300; //300ml

    @Override
    public void adeMake() {
        System.out.println(this.toString() + " 를 준비합니다.");
        System.out.println("포도를 씻고, 껍질은 제거합니다.");
        System.out.println("믹서기에 재료를 넣어 갈아줍니다.");
        System.out.println("얼음과 함께 컵에 담습니다.");
    }

    public String toString(){
        return "설탕 : " + SUGAR + "g " + "포도 : " + Grape + "개 " + "탄산수 : " + SPARKLING_WATER + "ml";
    }
}

public class DiTest {
    public static void main(String[] args) {
        Order order = new AdeMaker();
        order.adeOrder();
    }
}

실행결과 :
설탕 : 20g 딸기 : 5개 탄산수 : 300ml 를 준비합니다.
딸기를 씻고, 꼭지는 떼어냅니다.
믹서기에 재료를 넣어 갈아줍니다.
얼음과 함께 컵에 담습니다.
고객에게 음료를 전달합니다.

```

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/192087226-61988af1-0073-4969-894c-4913ea43ba57.png" width = 70%>
  </p>

코드는 위 구조를 나타내며, AdeMaker가 AdeRecipe와 StrawBerryAde를 의존하고 있는 형태이다.  
 위 코드는 큰 문제가 있다. AdeMaker는 딸기에이드밖에 만들지 못하기 때문이다. 포도에이드를 만들기 위해서는 class 내부의 new StrawBerry()를 new GrapeAde()로 수정을 해야한다. 하지만 이러면 이번엔 포도에이드 밖에 만들지 못한다. 또, AdeMaker가 딸기에이드르 참조하고 있을때, 딸기에이드가 잘 팔리지 않아서 없애버린다면? 코드를 수정하지 않으면 오류가 발생할 것이다.  
 이렇게 직접적으로 의존관계를 맺는 것은 알맞지 않는 코드이다.

이를 해결하는 코드로 바꿔보자.

```java
  private AdeRecipe ade;

  public AdeMaker(AdeRecipe ade){
      this.ade = ade;
  }
```

AdeMaker 클래스 구조를 위처럼 생성자를 통해 의존성(의존관계)을 외부에서 주입받게끔 수정하였다.  
그리고, 해당 객체를 만들때 어떠한 에이드로 만들지에 대한것을 정의해줘야 한다.

```java
  public static void main(String[] args) {
      Order order = new AdeMaker(new StrawBerryAde());
      order.adeOrder();

      System.out.println();

      order = new AdeMaker(new GrapeAde());
      order.adeOrder();
  }

실행결과 :
설탕 : 20g 딸기 : 5개 탄산수 : 300ml 를 준비합니다.
딸기를 씻고, 꼭지는 떼어냅니다.
믹서기에 재료를 넣어 갈아줍니다.
얼음과 함께 컵에 담습니다.
고객에게 음료를 전달합니다.

설탕 : 30g 포도 : 7개 탄산수 : 300ml 를 준비합니다.
포도를 씻고, 껍질은 제거합니다.
믹서기에 재료를 넣어 갈아줍니다.
얼음과 함께 컵에 담습니다.
고객에게 음료를 전달합니다.
```

위처럼 생성자의 파라미터로 어떠한 객체와 의존관계를 맺을지에 대한 정보를 전달해줘야 한다.  
이러면 AdeMaker의 입장에서는 코드를 손보지 않아도 다양한 에이드를 만들 수 있게 되었다.  
OCP(확장에는 열려있으며, 변경에는 닫혀있는 객체지향 원칙)를 지키게 된 것이다.

위의 방식은 객체를 만들때마다 매번 어떤 것을 주입할지 정의해야한다. 하지만 전반적으로 사용될 객체들을 하나의 클래스에 정의해두고 호출만 하도록하면, 해당 클래스만 관리해주면 되기 때문에 더욱더 편리하다.

```java
class AppConfig {
    public AdeMaker adeMaker(){
        return new AdeMaker(this.strawBerryAde());
    }

    public StrawBerryAde strawBerryAde (){
        return new StrawBerryAde();
    }
}

  public static void main(String[] args) {
      AppConfig appConfig = new AppConfig();
      Order order = appConfig.adeMaker(); //사용자의 입장에서 어떠한 에이드를 만들지에 대해 몰라도 됨
      order.adeOrder();
  }

```

위처럼 프로그램 내에서 외부에서 주입할 객체들을 관리할 클래스를 하나 만들면 main메서드에서처럼 사용하는 사용자 입장에서는 어떠한 객체를 넘길지에 대해 몰라도 사용할 수 있으며, 수정할 경우에도 Appconfig 클래스만 수정하면 된다.

### **_DI 정리_**

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/192086938-eba0c802-12b7-40d3-a108-40979de2302e.png" width = 70%>
  </p>

최종적으로 위처럼 표현이 가능하다. 위의 정적 클래스로만 보면 AdeMaker가 AdeRecipe를 의존하는 관계이긴 하지만, 해당 관계만으로는 AdeMaker가 어떠한 Ade의 구현 객체가 주입될지는 런타임에서 결정이 된다.

이렇게 **_애플리케이션 실행 시점에 외부에서 실제 구현 객체를 생성하여 동적인 의존 관계가 만들어지는 것을 DI라 한다._**

</br>

---

## **_IOC 컨테이너_**

위의 AppConfig 클래스가 바로 IOC 컨테이너의 기능을 간략히 흉내낸거라고 봐도 무방하다. 결국 개발자는 클래스와 메서드만 Spring의 표준에 맞게 비즈니스 로직에만 집중해서 개발을 하고, 그 외의 객체 생성, 의존 관계 주입, 제어 등은 Spring이 알아서 빈이라는 이름으로 객체를 만들어서 IOC(DI)컨테이너에 담아두고, 이후 요청에 따라 필요한 객체들을 컨테이너에서 꺼내서 동적으로 주입시켜서 사용하는 것이다.

</br>

---

## **_추가로 공부할 내용_**

- DI 3가지 방법(Field, Setter, Constructor)
- IOC 컨테이너의 동작원리
- Bean이란? (Bean이 생성되는 원리 + Bean 생명 주기)

</br>

---

# **_참고_**

- https://steady-coding.tistory.com/458 [제이온님의 IOC와 DI]
- https://somuchthings.tistory.com/129 [jeidiiy님의 IOC와 DI]
