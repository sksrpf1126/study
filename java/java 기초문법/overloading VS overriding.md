# **_overloading VS overriding_**

- ## **_overloading 예제_**

```java
public class OverLoadingEX {
    public static void main(String[] args){
        System.out.println(CalCulator.add(10, 20));
        System.out.println(CalCulator.add(10, 30L));
        System.out.println(CalCulator.add(10.0, 20.0));
    }
}

class CalCulator{
    static int add(int a, int b){
        return a + b;
    }

    static long add(int a, long b){
        return (long)a + b;
    }

    static double add(double a, double b){
        return a + b;
    }
}
```

오버로딩은 기존에 없는 새로운 메서드를 정의하는 것으로써, 메서드의 이름은 동일하지만 넘겨주는 파라미터의 개수나 타입은 다른 것이다.  
위 예제에서 add라는 메서드들은 각기 새롭게 정의한 메서드로써, 넘겨주는 파라미터의 타입에 따라 실행되는 메서드가 다르다.

대표적으로 자주쓰는 println 메서드 또한 호출할 때 넘겨주는 파라미터에 따라서 그에 맞는 메서드가 실행이 되는 것이다.

</br>

---

- ## **_overriding 예제_**

```java
public class OverridingEx {
    public static void main(String[] args){
        Point point = new Point();
        point.pointPrint();

        Point3D point3D = new Point3D();
        point3D.pointPrint();
    }
}

class Point{
    static int x = 10;
    static int y = 20;

    void pointPrint(){
        System.out.println("x : " + Point.x + " y : " + Point.y);
    }
}

class Point3D extends Point{
    static int z = 30;
    void pointPrint(){
        System.out.println("x : " + Point.x + " y : " + Point.y + " z : " + Point3D.z);
    }
}
```

오버라이딩은 상속관계에서 사용이 되며, 상속받은 메서드의 **_내용_** 을 변경하는 것이다. (오버로딩은 새롭게 **_정의_** 해서 만드는 것)  
메서드의 이름, 매개변수, 반환타입이 일치해야 하고 메서드의 내용만을 변경하는 것이다.  
위 예제와 같이 Point와 Point3D는 상속관계이며 pointPrint의 메서드를 Point3D 클래스에서 오버라이딩하여 메서드 내의 내용을 변경하였다.

오버라이딩은 메서드의 내용(동작)뿐 아니라 접근 제어자, 예외 또한 특정 조건에만 만족하면 변경할 수 있다.

조건1. 접근 제어자는 부모 클래스의 메서드보다 좁은 범위로는 변경이 불가하다.  
조건2. 예외는 부모 클래스의 메서드보다 더 많은 수를 선언할 수 없다.
