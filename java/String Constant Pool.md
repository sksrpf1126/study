# **_String Constant Pool_**

- ## **_문자열 선언 방법_**

  JAVA에서는 문자열 선언 방식이 2가지가 존재한다.

  ```java
  String literalString = "JAVA"; //리터럴

  String objectString = new String("JAVA"); //객체
  ```

  리터럴 방식과 객체를 통해 만드는 방식으로 나뉘는데 위 2가지 방식에는 실제 값은 "JAVA"로 동일하지만 메모리에 들어가는 방식이 다르기 때문에 값의 접근 방식 또한 다르다.

  리터럴 방식으로 선언한 문자열 변수는 Heap영역의 String Constant Pool(문자열 상수 풀)공간에 저장이 된다. (JAVA7에서 Perm 영역에서 Heap 영역으로 이동)

  이와 달리, 객체 방식의 경우에는 Heap영역에 저장이 된다.

  또 다른 차이점으로는 리터럴의 경우에 이미 문자열 상수 풀에 같은 값이 존재하는 경우에는 새로운 메모리 공간을 만들지 않고 이미 존재하는 값의 메모리 공간을 참조한다.  
  객체의 경우에는 값이 같더라도 Heap영역에 새로운 메모리 공간을 할당하여 객체를 담는다.

</br>

---

- ## **_== 연산자와 equals()_**

  문자열에서 == 연산자로 비교할 시에는 주소값을 비교하고, equals()는 주소가 가지는 실제값으로 비교한다.

  ```java
  String literalString1 = "JAVA";
  String literalString2 = "JAVA";

  if(literalString1 == literalString2){
      System.out.println("주소값이 일치합니다."); //실행
  }else{
      System.out.println("주소값이 일치하지 않습니다.");
  }

  if(literalString1.equals(literalString2)) {
      System.out.println("실제값이 일치합니다."); //실행
  }else{
      System.out.println("실제값이 일치하지 않습니다.");
  }
  ```

  위는 리터럴 방식으로 선언된 변수간 == 과 equals로 비교하는 것인데, 위에서 설명했듯이 리터럴로 선언된 문자열의 경우 문자열 상수풀로 저장이 되고, 이미 같은 값이 있는 경우에는 새로운 메모리 공간을 만들지 않고 이미 존재하는 메모리 공간을 참조한다.  
  그렇기에 위의 실행결과는 주소값과 실제값 둘다 일치한다고 반환한다.

  </br>

  ```java
  String ObjectString1 = new String("JAVA");
  String ObjectString2 = new String("JAVA");

  if(ObjectString1 == ObjectString2){
      System.out.println("주소값이 일치합니다.");
  }else{
      System.out.println("주소값이 일치하지 않습니다."); //실행
  }

  if(ObjectString1.equals(ObjectString2)) {
      System.out.println("실제값이 일치합니다."); //실행
  }else{
      System.out.println("실제값이 일치하지 않습니다.");
  }
  ```

  위는 두개의 문자열 객체를 만드는 것으로, 두 변수다 Heap영역에 각각의 메모리 공간을 할당하여 저장이 된다. 그렇기에 주소값은 일치하지 않고 실제값만이 일치한다.

  </br>

  ```java
  String literalString1 = "JAVA";
  String ObjectString1 = new String("JAVA");

  if(literalString1 == ObjectString1){
      System.out.println("주소값이 일치합니다.");
  }else{
      System.out.println("주소값이 일치하지 않습니다."); //실행
  }

  if(literalString1.equals(ObjectString1)) {
      System.out.println("실제값이 일치합니다."); //실행
  }else{
      System.out.println("실제값이 일치하지 않습니다.");
  }
  ```

  리터럴로 선언한 문자열 변수의 경우에는 문자열 상수풀에 저장이 되고, 객체의 경우에는 Heap영역에 저장이 된다. 그렇기에 당연히 주소값은 다르며, 실제값은 일치하는 것이다.

  정리하면, 위와 같이 == 연산자로는 문자열간의 값을 비교하는게 아닌 주소값을 비교하는 것이기 때문에 값이 같다고해도 == 이 false가 될 수 있는 것이다. 그러므로 문자열간의 값을 비교할 때에는 equals() 메서드를 통해 비교해야 한다.

</br>

---

# **_참고_**

- https://hyeran-story.tistory.com/123 [equals 와 == 차이점]
- https://madplay.github.io/post/java-string-literal-vs-string-object [String Constant Pool]
