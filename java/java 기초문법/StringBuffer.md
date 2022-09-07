# **_StringBuffer_**

## **_StringBuffer란?_**

일반적인 String 클래스의 경우에는 값을 변경할 수 없다. 단지 값을 변경하면 새로운 메모리 공간을 할당하여 해당 값을 넣는것이다.  
하지만 StringBuffer는 메모리 공간을 만들지 않고 값 변경이 가능하다는 것이다.

</br>

---

## **_StringBuffer 크기_**

StringBuffer 또한 String 클래스와 마찬가지로 내부적으로는 char[] 배열 형태로 문자열이 저장이 된다.

StringBuffer는 char[]배열의 크기를 생성자를 통해 지정할 수 있다.

```java
StringBuffer sb1 = new StringBuffer(); // 기본생성자 호출시 16크기

StringBuffer sb2 = new StringBuffer(100); // 100크기

StringBuffer sb3 = new StringBuffer("123"); // 문자열 크기 + 16크기
```

위와 같이 생성자에 넘겨주는 파라미터에 따라서 크기가 달라진다.

</br>

---

## **_StringBuffer 메모리 동작원리_**

```java
StringBuffer sb1 = new StringBuffer("123");
StringBuffer sb2 =  sb1.append("zzz");

System.out.println(sb1);
System.out.println(sb2);

sb1.append("abc");

System.out.println(sb1);
System.out.println(sb2);

실행 결과 :
123zzz
123zzz
123zzzabc
123zzzabc
```

StringBuffer.append()의 리턴 값은 StringBuffer의 주소이다.  
그래서 위에 "123"이 저장되어있는 인스턴스를 참조하는 sb1에 "zzz"를 추가하면서 반환된 인스턴스의 주소값을 sb2에 할당시키는 것.  
이를 통해 sb1 과 sb2 의 참조변수는 같은 인스턴스를 가르키게 된다. 이후 sb1을 통해 참조하고 있는 인스턴스에 값을 추가하면 sb2 또한 값이 같이 바뀌게 되는 것이다.

</br>

```java
StringBuffer sb1 = new StringBuffer("가");
sb1.append("나").append("다");
```

append는 StringBuffer의 주소, 즉 인스턴스의 주소를 반환하기 때문에 위와 같이 메소드체이닝기법(반환된 값으로 새로운 메서드를 연속 호출)으로 구현이 가능

</br>

---

## **_StringBuffer Equals()_**

```java
StringBuffer sb1 = new StringBuffer("java");
StringBuffer sb2 = new StringBuffer("java");

System.out.println(sb1 == sb2);
System.out.println(sb1.equals(sb2));

StringBuffer sb3 = sb1.append("abc");

System.out.println(sb1 == sb3);
System.out.println(sb1.equals(sb3));

실행결과 :
false
false
true
true
```

String 클래스는 equlas()가 오버라이딩이 되어있기 때문에 값을 비교하는 메서드로 사용이 되지만, StringBuffer 클래스는 오버라이딩이 되어있지 않기 때문에 객체의 주소 즉 == 연산자와 동일하게 작동이 된다.

위처럼 sb1, sb2 각각 StringBuffer의 인스턴스를 만들어서 == 과 equals()로 비교해보면 false로 나온다.

반대로 sb1에 대한 인스턴스의 주소값을 sb3에 넣어서 비교해보면 true가 나온다.

</br>

---

## **_StringBuilder_**

StringBuffer는 Thread 환경에서 안전하도록 "동기화"가 되어 있다. 그러다 보니 StringBuffer는 멀티쓰레드 환경이 아닌 경우에서는 이 동기화가 성능을 떨어트리는데, 이 때 나온 클래스가 StringBuilder이다.

StringBuilder는 StringBuffer와 이름만 다를 뿐 거의 동일하게 사용하면 된다. Thread에서의 동기화가 없기 때문에 멀티쓰레드에서의 안전은 확보가 안되겠지만 성능은 향상이 되었다.
