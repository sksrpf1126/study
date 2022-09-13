# **_Hashset_**

Set인터페이스를 상속받는 컬렉션 클래스 중 하나로, 데이터 순서 보장 X, 중복 데이터 허용 X 라는 특징을 지니고 있다.

데이터 순서를 보장받고 싶을 경우 LinkedHashSet 클래스를 사용하면 된다.

```java
public class HashSetEx1 {
    public static void main(String[] args) {
        Object[] objArr = {"1", new Integer(1), "2", "2", "3", "3", "4", "4"};
        HashSet hs = new HashSet();

        for(int i=0; i < objArr.length; i++){
            hs.add(objArr[i]);
        }

        System.out.println(hs);

        Iterator it = hs.iterator();

        while(it.hasNext()){
            Object itObj = it.next();
            System.out.println("value : " + itObj + ", type : " + itObj.getClass().getName());
        }
    }
}

실행결과 :
[1, 1, 2, 3, 4]
value : 1, type : java.lang.String
value : 1, type : java.lang.Integer
value : 2, type : java.lang.String
value : 3, type : java.lang.String
value : 4, type : java.lang.String
```

1 두개가 들어간 것처럼 보이지만, 타입까지 찍어보면 String과 Integer 즉, 타입이 다르다는 것을 알 수 있다.

```java
public class HashSetEx2 {
    public static void main(String[] args) {
        Set set = new HashSet();

        for(int i = 0; set.size() < 6; i++){
            int num = (int)(Math.random() * 45) + 1;
            set.add(num); // new Interger(num) 오토박싱, 중복값이면 저장안되고, size 증가도 안됨
        }

        List list = new ArrayList(set); // new LinkedList(list)도 가능
        Collections.sort(list); //메서드의 인자로 List 타입을 필요로 하기 때문에 위에서 변환
        System.out.println(list);
    }
}

실행결과 :
[9, 10, 16, 34, 43, 45]
```

복권 6개 숫자를 뽑는 기능을 구현한 것으로써, set의 특징은 중복 허용 X를 활용한 예제이다.

set.size()가 6이 되기전까지 반복문을 실행시키는데, 중복은 들어가지 않으므로 size또한 변경되지 않는다. 그래서 무조건 중복없는 6개의 숫자를 뽑을 수 있는 것이다.

Collections 클래스(Collection 인터페이스와 다름)의 sort기능을 활용하여 정렬하였는데 Collections.sort는 파라미터로 List를 받기때문에 HashSet을 ArrayList로 형변환 후에 정렬을 시킨것이다.

```java
public class HashSetEx3 {
    public static void main(String[] args) {
        HashSet set = new HashSet();
        set.add("abc");
        set.add("abc");
        set.add(new Person(10,"홍길동"));
        set.add(new Person(10,"홍길동"));

        System.out.println(set);
    }
}

class Person {
    int age;
    String name;

    public Person(int age, String name){
        this.age = age;
        this.name = name;
    }

    public String toString(){
        return this.name + ":" + this.age;
    }

}

실행결과 :
[홍길동:10, abc, 홍길동:10]
```

Set 인터페이스를 상속받는 컬렉션 클래스들이 **_값_** 은 종북이 되지 않지만, 값이 같지만 서로다른 인스턴스의 경우에는 서로다른 객체라고 판단하여 값을 저장한다.

그렇기에 위처럼 결과가 나타난다.

이를 해결하기 위해서는 Person 클래스에

```java
    @Override
    public boolean equals(Object o) {
        if (this == o) return true; //같은 객체(인스턴스)라면 바로 true
        if (o == null || this.getClass() != o.getClass()) return false; //null 또는 class가 다르면 false (비교 대상이 아님)
        Person person = (Person) o; //비교하기 전 형변환
        return this.age == person.age && Objects.equals(name, person.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(age, name);
    }
```

equals와 hasCode를 오버라이딩을 해야하는데 그 이유는, HashSet의 add 메서드는 요소를 추가하기전에 **_"요소"_** 의 equals메서드와 hasCode 메서드를 내부적으로 호출하여 중복여부를 판별하기 때문이다.

그래서 위처럼 Person 클래스에 오버라이딩하고 나서, 호출하고 나면 결과가 [abc, 홍길동:10] 로 서로다른 인스턴스여도 값의 중복을 체크하여 중복을 허용하지 않는다.
