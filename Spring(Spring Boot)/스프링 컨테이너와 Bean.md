# **_스프링 컨테이너와 빈_**

IOC와 DI를 공부하면서, 스프링 컨테이너와 빈이 꼬리를 무는 내용으로 등장하였다. 그 이유는 스프링 컨테이너가 IOC와 DI의 원리가 적용되기 때문이다.  
그리고 스프링 컨테이너가 관리하는 객체들을 **_"Bean(빈)_** 이라는 이름으로 부른다.

</br>

---

## **_빈을 관리하는 방법들_**

스프링이 아닌 스프링부트에서는 어노테이션을 통해 애플리케이션의 실행과 동시에 빈이 자동으로 등록이 된다. 그렇기에 스프링부트만을 얕게 사용해본 나로써는 그저 넘어가는 개념 중 하나였지만 스프링(부트)를 제대로 사용하기 위해서 필요한 핵심적인 개념이라는 것을 최근에 알게 되었고, 우선 빈을 관리하는 방법들부터 볼려고 한다.

SpringBoot 이전에는 XMl을 통해 Bean을 관리하는 방법과 자바코드로 이루어진 설정파일을 통한 Bean을 관리하는 방법으로 크게 나눠진다.

Xml을 통해 Bean을 관리하는 방법에 대해서는 https://wordbe.tistory.com/entry/Spring-IoC-%EB%B9%88-%EB%93%B1%EB%A1%9D-%EB%B0%A9%EB%B2%95-5%EA%B0%80%EC%A7%80 에 코드가 나와있다. (최근에는 잘 사용이 되지 않는 방법이다.)

그리고 자바 설정파일로 Bean을 관리하는 방법에 대해서는 코드로 정리한다.  
해당 코드는 IOC와 DI의 코드와 이어지는 내용이다.

```java
@Configuration
public class AppConfig {
    @Bean
    public AdeMaker adeMaker(){
        return new AdeMaker(this.strawBerryAde());
    }

    @Bean
    public StrawBerryAde strawBerryAde (){
        return new StrawBerryAde();
    }
}
```

AppConfig 클래스에 @Configuration 어노테이션과 메서드마다 @Bean 어노테이션을 붙였다.  
우선적으로 @Bean이 붙으면 스프링 컨테이너에게 해당 객체를 Bean으로 관리해달라고 선언한 것이다.  
일반적인 사용용도는 개발자가 컨트롤이 불가능한 외부 라이브러리들을 빈으로 등록할 때 사용이 된다.

다음으로 @Configuration이다. 해당 어노테이션이 붙어있지 않는다면, @Bean이 붙어있어도 스프링 컨테이너는 감지할 수가 없다.(설정을 건드리면 가능하다는데, 추천하는 방식은 아닌듯하다)  
그 이유는 @Configuration 내부를 들여다보면 @Component가 붙어있는데, 스프링 컨테이너는 @ComponentScan을 통해서 @Component가 붙어있는 모든 것들을 빈으로 등록해주기 때문이다.

빈으로 등록이 이루어 졌다면 이를 가져와서 사용해보도록 하자.

```java
public class HelloSpringApplication {

	public static void main(String[] args) {
		BeanFactory beanFactory = new AnnotationConfigApplicationContext(AppConfig.class);
		Order order = beanFactory.getBean("adeMaker",AdeMaker.class);
		order.adeOrder();
	}
}

실행결과:
15:43:49.490 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@3891771e
15:43:49.512 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
15:43:49.660 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerProcessor'
15:43:49.663 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerFactory'
15:43:49.665 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalAutowiredAnnotationProcessor'
15:43:49.667 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalCommonAnnotationProcessor'
15:43:49.679 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'appConfig'
15:43:49.687 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'adeMaker'
15:43:49.703 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'strawBerryAde'
설탕 : 20g 딸기 : 5개 탄산수 : 300ml 를 준비합니다.
딸기를 씻고, 꼭지는 떼어냅니다.
믹서기에 재료를 넣어 갈아줍니다.
얼음과 함께 컵에 담습니다.
고객에게 음료를 전달합니다.
```

AnnotationConfigApplicationContext에 파라미터로 참고할 설정파일을 정의해주면, 스프링 컨테이너에 빈들이 등록이 된다.(componentScan이 아닌 직접적인 코드로 등록)  
이후 등록된 빈을 가져와서 사용하는 코드이다.

실행결과에서 주의깊게 봐야할 부분이 있다.

```java
15:43:49.679 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'appConfig'
15:43:49.687 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'adeMaker'
15:43:49.703 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'strawBerryAde'
```

이 부분으로 AppConfig 뿐만 아니라 AppConfig에서 @Bean으로 정의한 부분들까지 전부 빈으로 등록이 된것을 확인할 수 있다.

```java
BeanFactory beanFactory -> ApplicationContext beanFactory
```

위와 같이 참조타입만 변경하여도 결과가 똑같은 것을 확인할 수 있는데, 그 이유는 ApplicationContext가 내부적으로 BeanFactory를 상속받기 때문이다.

- ### **_BeanFactory VS ApplicationContext_**

  BeanFactory는 빈에 대한 관리의 기능이 들어있지만, ApplicationContext는 추가적으로 국제화가 지원되는 텍스트 메시지 관리, 이미지같은 파일 자원을 로드, 리스너로 등록된 빈에게 이벤트 발생 알림 등 부가적인 기능까지 추가되어 있다.

</br>

위 코드는 XML이 아닌 자바 코드로 이루어진 설정파일로 빈을 직접 등록하는 방법이다.  
하지만 지금의 방식은 어노테이션으로 위에서 잠깐 설명한 @ComponentScan을 통해 @Component 어노테이션이 붙은 클래스들을 찾아서 자동으로 빈을 등록해주기 때문에 위처럼 쓸 명확한 이유가 있지 않은 이상은 수동으로 빈을 등록할 이유는 거의 없다.

@Controller, @Service, @Repository 와 같은 어노테이션들도 내부를 보면 @Component가 있다. 즉 해당 어노테이션들이 붙으면 자동으로 스프링 컨테이너에 빈으로 등록이 되어서 IOC에 의해 스프링이 DI를 해준다!

</br>

---

## **_Bean은 Singleton으로 관리_**

빈은 싱글톤패턴을 이용하여 단 하나의 객체만을 만들어서 관리한다.  
어노테이션으로 자동으로 빈으로 등록되는 것들은 알아서 싱글톤으로 지정하여 만들지만, 수동으로 등록하는 빈들은 어떻게 싱글톤으로 만들어질까?

해답은 @Configuration에 있다.  
해당 어노테이션이 붙어서 @Bean으로 등록되는 경우에는 싱글톤으로 만들어지는데, 원리는 CGLIB 내장 라이브러리(바이트 코드를 조작하는 라이브러리)를 통해서이다.

@Configuration이 붙은 클래스는 CGLIB에 의해 해당 클래스를 상속받는 임의의 클래스를 하나 만든다. 그리고 임의의 클래스에서 @Bean으로 등록된 코드들에서 똑같은 객체를 만들려고 할 때 해당 객체가 빈으로 등록되어있는지에 대해 여부를 먼저 확인한 후에 없을 때만 만드는 것이다.

```java
@Configuration
public class AppConfig {
    @Bean
    public AdeMaker adeMaker(){
        return new AdeMaker(this.strawBerryAde());
    }

    @Bean
    public StrawBerryAde strawBerryAde (){
        return new StrawBerryAde();
    }
}
```

위 코드를 보면 Bean으로 등록되는 것은 AppConfig 그리고 adeMaker, strawBerryAde 이다.(빈의 이름은 메서드 이름으로 만들어 진다)  
여기서 주의 깊게 봐야하는 부분은 adeMakder 내부에서 this.strawBerryAde() 부분이다. 해당 메서드를 통해서 StrawBerryAde 객체를 만들려고 시도를 하는 것이다.  
하지만 CGLIB에 의해서 이미 strawBerryAde가 빈으로 등록되어 있기 때문에 새로 만들지 않고 해당 빈으로 주입시키는 것이다.

</br>

---

## **_@Configuration + @Bean VS @Component + @Bean_**

수동으로 빈을 주입하는 방법에는 지금까지 설명한 @Configuration + @Bean의 방법과 또 다른 방법으로는 @Configuration 대신에 @Component 어노테이션을 활용하는 것이다.  
빈을 주입하는 것은 똑같지만 객체를 어떻게 생성하고 유지하냐는 관점에서 다르다.  
@Configuration + @Bean 은 CGLIB에 의해서 싱글톤으로 빈이 관리가 되지만 @Component로 바꿔서 빈을 생성하는 경우에는 CGLIB은 실행이 되지 않는다.  
결국, 싱글톤으로 객체를 만들고 생성하지 않고, 호출할때마다 계속 새로운 객체를 만들어서 반환한다. 이를 방식을 Bean Lite Mode라고 한다.

객체를 싱글톤(싱글톤 빈)으로 유지할 것인지, 아님 새로운 객체(프로토타입 빈)를 만들어서 사용할 것인지는 사용용도에 따라 개발자가 판단을 해야한다.  
기본적으로 읽는 용도와 상태가 없는 공유 객체로써 사용이 이루어지는 경우에는 싱글톤이 맞지만, 쓰기가 가능하고 사용할 때마다 상태에 변화를 주고 해당 상태를 활용해야한다면 프로토타입으로 만드는 것이 옳은 판단일 것이다.

</br>

---

## **_수많은 요청을 하나의 Controller로 처리가 가능한 이유_**

지금까지 스프링에서는 빈이라는 객체로 싱글톤을 통해 하나만을 생성하여 사용한다.  
수백, 수천의 같은 요청이 들어올 경우에는 그럼 어떻게 사용이 되는가?  
스프링 컨테이너와 빈을 공부하면서 생긴 의문인데 나보다 먼저 같은 고민을 한 사람들의 글을 읽고 정리를 해보고자 한다.  
다양한 WAS 중 많이 쓰는 Tomcat은 쓰레드 풀에서 보통 200개의 쓰레드까지 생성하여 작업한다.  
그러니까 수백, 수천의 요청이 들어와도 일단 200개정도의 작업을 먼저 할당하여 처리한다는 것이다. 하지만 그래도 200개의 같은 요청을 처리해야한다면, 과연 어떻게 될까?  
해답은 JVM 메모리구조에 있다. 객체는 만들 때 힙영역으로, 그리고 객체를 만들기 위해 필요한 클래스에 대한 정보(클래스 내의 메서드도 포함)는 메서드영역으로 저장이 된다.

힙영역과 메서드영역은 모든 쓰레드가 공유하는 공간인데, 그럼 200개의 쓰레드에서 동시에 같은 객체의 메서드를 사용하기 위해서 접근하는 건가? 그럼 동기화는? 이런 생각을 가지고 있었는데, https://jeong-pro.tistory.com/204 에서 나의 고민을 해결해주는 글을 발견하였다.

간단한거였는데 너무 복잡하게 생각한 부분이었다. 쓰레드를 공부할 때 보통 동기화를 하는 이유가 무엇인가? 자원을 동시에 작업이 발생할 경우에 문제가 발생할 수 있기에 동기화를 하는 것이다.  
근데 스프링에서는 컨트롤러와 컨트롤러에서 또 호출하는 서비스클래스의 메서드는 결국 메서드를 호출할 뿐이지 특정 자원을 변경하는 작업이 아니다.  
그렇기에 그저 **_공유_** 의 입장에서 모든 쓰레드가 하나의 객체의 메서드를 호출할 뿐이라는 것이다.

### **_+ 그럼 싱글톤은 Thread-Safe 한가?_**

아니다. 하나의 객체만을 만들 수 밖에 없을 뿐 해당 객체에 자원을 변경하는 행위와 같은 동작이 있을 경우에는 문제가 발생한다.  
그렇기에 싱글톤 빈은 읽기용도로만 사용해야지, 상태를 변화하는 행위와 같은 행위는 하지 않아야 하며 해야한다 해도 쓰레드 동기화를 고려해서 코드를 짜야한다는 것이다.

그리고 컨트롤러나 서비스 메서드 내에서 또다른 빈을 사용할 경우에도 해당 빈(객체)에서도 멀티쓰레드 환경을 고려해서 코드를 짜야한다.

</br>

---

## **_추가로 공부할 내용_**

- 컨트롤러나 서비스 클래스의 메서드 내에서 객체(빈)내의 자원을 변경하는 코드가 있을 경우 여러 요청이 동시에 들어온다면 멀티쓰레드로 동작이 되어 문제가 발생하는가?
- 빈을 주입해서 사용하는 방법(field, setter, constructor 3가지)

</br>

---

# **_참고_**

- https://jeong-pro.tistory.com/204 (수많은 요청을 하나의 Controller가 처리할 경우)
- https://steady-coding.tistory.com/459 (스프링 컨테이너와 빈)
- https://steady-coding.tistory.com/594 (스프링 빈 정리)
