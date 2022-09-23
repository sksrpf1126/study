# **_Stream_**

수많은 데이터를 다룰때 배열 또는 컬렉션 클래스들을 활용하여 for문과 Iterator를 통해 요소에 접근하여 사용하였다.  
하지만, 이러면 코드가 길어지기도 하고 재사용성이 떨어지기도 한다.

또 다른 문제는 데이터 소스마다 다른 방식으로 다뤄야 한다는 것이다. 컬렉션 클래스들은 그나마 Iterator를 통해 표준화하였지만, 각 컬렉션 클래스에는 같은 기능의 메서드들이 중복되서 정의되어 있으며, 배열과도 동일하다.  
Collections.sort 와 Arrays.sort가 이에 해당한다.

이러한 문제를 해결하기 위해 등장한 것이 **_Stream_** 이다.

</br>

---

## **_Stream 특징_**

- 스트림은 데이터소스를 변경하지 않는다.  
  데이터 소스를 통해 스트림을 만드는데, 이 때 스트림은 해당 데이터 소스의 데이터만 읽어서 활용할 뿐이지 데이터 소스 자체(원본)에는 아무런 변경도 할 수 없다.(하지 않는다.)

- 스트림은 일회용이다.  
  간단하게 말하면 Iterator와 동일하다.  
  한번 만들고 사용하면, 다시 만들지 않는 이상 재사용이 불가능하다.(정확히 말하면 중간 연산은 여러번 가능하지만 최종 연산이 진행된 이후에는 사용 불가)

- 스트림은 작업을 내부 반복으로 처리한다.  
  배열같은 경우 for문이라는 반복문을 개발자가 직접 작성하여 처리한다. 하지만 스트림의 경우에는 반복문을 메서드 안에 숨겼기 때문에 forEach에 파라미터로 어떠한 메서드를 호출할지만 넘겨주면 내부에서 반복이 이루어진다.

- 지연된 연산  
  중간 연산을 담당하는 메서드들을 아무리 호출하여도 연산이 수행되지 않는다. 최종 연산 메서드까지 호출해야지 해당 stream이 중간 연산 메서드들부터 접근하여 최종 연산까지 한번에 진행하게 된다.

- 이 외에도 병렬 스트림, IntStream, LongStream과 같은 오토박싱 언박싱의 비효율을 발생시키지 않기 위한 기본형을 다루는 스트림 제공이 존재한다.

</br>

---

## **_Stream 만들기_**

</br>

```java
public class StreamEx1 {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1,2,3,4,5);
        Stream<Integer> intStream = list.stream();

        intStream.forEach(System.out :: println);
//        intStream.forEach((str) -> System.out.println(str));
    }
}

실행결과:
1
2
3
4
5
```

위는 컬렉션 클래스 중 하나인 ArrayList를 Stream으로 변환한 것이다.  
위 forEach문은 주석친 부분과 동일하게 동작된다.

</br>

```java
public class StreamEx2 {
    public static void main(String[] args) {
        Stream<String> strStream1 = Stream.of("a","b","c");
        Stream<String> strStream2 = Stream.of(new String[]{"a","b","c"});

        String[] strArray = {"a","b","c"}; // String[] strArray = new String[]{"a","b","c"};

        Stream<String> strStream3 = Stream.of(strArray);
        Stream<String> strStream4 = Arrays.stream(new String[]{"a","b","c"});
    }
}
```

다른 방법으로 Stream을 만든 것이다.

</br>

---

## **_stream 연산(일부분)_**

</br>

```java
public class StreamEx3 {
    public static void main(String[] args) {
        IntStream intStream = IntStream.of(1,2,3,1,2,3,4,5,6,7);
        intStream.distinct().forEach(System.out :: println);

        System.out.println("--------------");

        intStream = IntStream.of(1,2,3,1,2,3,4,5,6,7);
        intStream.filter((i) -> i % 2 == 0).forEach(System.out :: println);
    }
}

실행결과:
1
2
3
4
5
6
7
--------------
2
2
4
6
```

위에 distinct와 filter는 중간연산 메서드이고, forEach는 최종연산 메서드이다.  
중간연산은 여러번 호출이 가능하며, 최종연산은 단 하나만을 사용할 수 있다.

위는 중복제거 또는 필터링을 하는 코드이다.

</br>

```java
public class StreamSortedEx {
    public static void main(String[] args) {
        Stream<Student> studentStream = Stream.of(
                new Student("이자바", 3, 300),
                new Student("김자바", 1, 200),
                new Student("안자바", 2, 100),
                new Student("박자바", 2, 150),
                new Student("소자바", 1, 200),
                new Student("나자바", 3, 290),
                new Student("김자바", 3, 180)
        );

        studentStream.sorted(Comparator.comparing(Student::getBan)
                .thenComparing(Comparator.naturalOrder()))
                .forEach(System.out :: println);
    }
}

class Student implements Comparable<Student> {
    private String name;
    private int ban;
    private int totalScore;

    Student(String name, int ban, int totalScore){
        this.name = name;
        this.ban = ban;
        this.totalScore = totalScore;
    }

    public String toString(){
        return String.format("[%s, %d, %d]", this.name, this.ban, this.totalScore);
    }

    public String getName() {
        return name;
    }

    public int getBan() {
        return ban;
    }

    public int getTotalScore() {
        return totalScore;
    }

    @Override
    public int compareTo(Student s) {
        return s.totalScore - this.totalScore;
    }
}

실행결과:
[김자바, 1, 200]
[소자바, 1, 200]
[박자바, 2, 150]
[안자바, 2, 100]
[이자바, 3, 300]
[나자바, 3, 290]
[김자바, 3, 180]
```

위는 Sorted라는 스트림의 중간 연산 메서드를 활용하여 정렬한 후 forEach문의 최종 연산 메서드로 출력한 결과를 보여주는 코드이다.

Sorted내부에서 처음에 호출한 것은 Comparator.comparing(Student :: getBan) 인데, 내부 코드를 보면

```java
    public static <T, U extends Comparable<? super U>> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor)
    {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));
    }
```

간단하게 보면, 결국 Student의 getBan을 통해 반한된 값으로 compareTo를 호출하여 비교를 진행하는 것이다.

그래서 반으로 우선 정렬이 이루어진 뒤에 추가적으로 .thenComparing(Comparator.naturalOrder())를 통해 Student에 오버라이딩된 compareTo를 호출하여 점수로 정렬하는 것이다.
