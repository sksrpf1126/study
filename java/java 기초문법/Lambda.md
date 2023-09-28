# **_Lambda_**

람다식은 간단히 말해서 메서드를 하나의 식으로 간단히 표현한 것이다.

메서드명과 반환값이 없어지므로, 익명 함수라고도 한다.(익명 함수보다는 익명 객체가 맞는 말)

객체지향 언어인 자바에서 메서드를 사용하기 위해서는 보통 클래스내에 메서드를 선언하고 객체를 만들어서 해당 객체를 통해 메서드를 호출한다.(static 이면 객체 필요 X)

하지만 람다식을 활용하면 클래스를 만들 필요 없이 메서드 사용이 가능하며, 또한 매개변수로도 전달이 가능하여 메서드 즉, 람다식을 변수처럼 사용이 가능하다.

기본적인 문법형태로는

```java
int max(int a, int b){return a > b ? a : b;}
```

를 람다식으로 바꾸면

```java
//1번째 형태 (메서드 명 제거)
(int a, int b) -> {return a > b ? a : b;}

//2번째 형태
//내용이 한줄일 경우에는 중괄호 생략이 가능하며(return 문 있을 시 return문을 같이 생략해야 한다.) 맨 끝에 세미콜론은 생략한다.
(int a, int b) -> a > b ? a : b

//3번째 형태
//가장 간결한 형태로 타입 추론이 가능한 경우에는 타입을 생략할 수 있다.
(a, b) -> a > b ? a : b
```

</br>

---

## **_람다식은 익명 함수? 익명 객체!_**

자바는 객체지향 언어이기 때문에 메서드를 사용하기 위해서는 클래스 내에 포함이 되어 있어야 한다.  
그럼 람다식 또한 결국은 어느 클래스 내에 포함이 되서 메서드로써 동작이 되는 것이다.

람다식은 사실, 익명 클래스의 익명 객체와 동등하다.

```java
(int a, int b) -> a > b ? a : b
```

위 람다식을 익명 클래스의 모양으로 바꿔보면

```java
new Object(){
  int max(int a, int b){
    return a > b ? a : b ;
  }
}
```

의 모양이다. 그럼 이러한 익명 클래스에 의해 만들어지는 익명 객체를 저장할 참조변수가 필요하다는 것인데, 그럼 어떠한 타입의 참조변수로 만들어야 저장이 가능할까?

- Obejct 클래스는 아니다. 위는 그저 익명클래스의 모양을 표현하기 위함일 뿐 실제로 Object로 만들어보면 에러가 난다.

객체지향 틀을 깨지 않으면서, 위의 문제를 해결하기 위해 등장한 것이 함수형 인터페이스이다.

함수형 인터페이스란 단 하나의 추상메서드만을 가지고 있는 인터페이스를 나타내며, @FunctionalInterface 애너테이션을 걸어둠으로써, 함수형 인터페이스의 규칙을 지킬 수 있다.

```java
interface MyFunction {
  public abstract int max(int a, intb);
}

//익명 객체 생성
MyFunction f = new MyFunction(){
  public int max(int a, int b){
    return a > b ? a : b;
  }

  int big =  f.max(3,5); //익명 객체를 참조하는 참조변수를 통해 메서드 호출
}
```

위를 람다식의 형태로 바꾸면

```java
MyFunction f = (a,b) -> a > b ? a : b;
int big = f.max(5,3); //익명 객체의 메서드 호출
```

위와 같이 간결하게 표현이 가능하다. 추상 메서드가 단 하나 뿐이기에 람다식 하나를 작성하면 자동으로 매칭이 되어지기 때문에 위처럼 작성이 가능하다.

</br>

---

## **_함수형 인터페이스 타입의 매개변수, 반환 타입_**

처음에 람다식에 대해 설명할 때 메서드를 하나의 변수로 사용이 가능하다 했다.

```java
package chapter14;

public class LambdaEx1 {
    static void execute(MyFunction f){
        f.run();
    }

    static MyFunction getMyFunction(){
        MyFunction f = () -> {
            System.out.println("f3.run()");
        };

        return f;
    }
    public static void main(String[] args) {
        MyFunction f1 = () -> {
            System.out.println("f1.run()");
        };

        MyFunction f2 = new MyFunction() {
            @Override
            public void run() {
                System.out.println("f2.run()");
            }
        };

        MyFunction f3 = getMyFunction();

        f1.run();
        f2.run();
        f3.run();

        execute(f1);
        execute(() -> {
            System.out.println("f4.run()");
        });

    }
}

@FunctionalInterface
interface MyFunction {
    void run(); // == public abstract void run();
}

실행결과:
f1.run()
f2.run()
f3.run()
f1.run()
f4.run()
```

처음찍힌 f1.run()의 경우에는 함수형 인터페이스 타입의 참조변수를 선언하여 해당 내부 추상 클래스를 람다식 즉, 익명 객체를 만들어 참조하게 만든 것이다.

f2.run()은 위 방식을 생략없이 표현한 문법이다.

f3.run()은 참조변수 형태로 반환하여 메서드를 만들어서 준 것이다.

4번째에 찍힌 f1.run()의 경우에는 참조변수를 전달하여서 해당 메서드를 호출한 것이다.

마지막 f4.run()의 경우에는 파라미터로 람다식을 줄 수 있다는 것을 표현한 것이다.

</br>

---

## **_람다식 형변환_**

함수형 인터페이스로 람다식을 참조할 수 있는 것일 뿐, 람다식의 타입이 함수형 인터페이스의 타입과 일치하는 것은 아니다.  
람다식은 익명 객체이고, 익명 객체는 타입이 없다. 정확히는 타입이 있지만 컴파일러가 임의의 이름을 정하기 때문에 알 수 없는 것이다. 그래서 대입 연산자의 양변의 타입을 일치시키기 위해 아래와 같이 형변환이 필요하다.

```java
MyFunction f = (MyFunction) (() -> {}); // 양변의 타입이 다르기때문에 형변환이 필요, 형변환 생략가능
```

람다식은 Object타입으로 형변환은 할 수없다.
그럼에도 Object타입으로 형변환을 하려면 함수형 인터페이스로 형변환을 먼저 해야한다.

```java
Object obj = (Object) (() -> {}); // Error, 함수형 인터페이스만 형변환 가능
Object obj = (Object) (MyFunction) (() -> {}); // 함수형 인터페이스로 먼저 형변환, Object타입으로 다시 형변환
String str = ((Object) (MyFunction) (() -> {})).toString();
```
