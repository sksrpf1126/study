# **_Call By Value VS Call By Reference_**

- ## **_Call By Reference_**

```C
  void callByReferenceTest(int *n){
    n = 20; //원본의 실제 데이터를 변경
  }

  void main(){
    int number = 10;
    callByReferenceTest(&number);
    printf("%d", number); //20
  }
```

- Call By Reference는 원본의 실제 값에 접근하여 변경이 가능하다.  
  그 이유는 Call By Value와 다르게 새로운 메모리 공간에 값을 복사하여 넘겨주는게 아닌 원본을 참조하는 형태이기 때문이다.  
  이를 통해 Call By Reference의 특징을 지닌 언어는 복사가 아닌 참조를 통하여 메모리 공간을 절약하였지만, 원본 데이터가 변경될 수 있다는 위험성이 존재한다.

  </br>

  ***

- ## **_Call By Value_**

```Java
class Cat {
    private String name;

    Cat(String name){
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

public class CallByValue {

    public static void main(String[] args) {

        int x = 1;
        int[] y = {1,2,3};
        Cat cat1 = new Cat("고양이1");
        Cat cat2 = new Cat("고양이2");

        System.out.println("x :  " + x); //1
        System.out.println("y[0] :  " + y[0]); //1
        System.out.println("cat1 name :  " + cat1.getName()); //고양이1
        System.out.println("cat2 name :  " + cat2.getName()); //고양이2

        callByValueTest(x, y, cat1, cat2);

        System.out.println("x :  " + x); //1
        System.out.println("y[0] :  " + y[0]); //2
        System.out.println("cat1 name :  " + cat1.getName()); //고양이1
        System.out.println("cat2 name :  " + cat2.getName()); //이름 바꾼 고양이2

    }

    static void callByValueTest(int x, int[] y, Cat cat1, Cat cat2){
        x++;
        y[0] = 10;
        cat1 = new Cat("이름 바꾼 고양이1"); //cat1은 복사본이기 때문에 cat1에 새로운 객체를 참조하게 한다하여도, main메서드 내의 원본인 cat1에서는 기존의 객체를 참조한다.
        cat2.setName("이름 바꾼 고양이2");  //원본의 데이터(객체의 주소값)가 아닌 원본이 참조하고 있는 객체의 프로퍼티에는 접근할 수 있음.
    }
}
```

- Call By Value는 위와 같이 값을 넘길 때에 복사본(새로운 메모리 공간)을 만들어서 값을 저장한다.  
  그래서 위와 같이 원본의 데이터에는 접근이 불가능하지만, 같은 객체의 주소 값을 참조하고있기 때문에 객체의 프로퍼티에는 접근이 가능하다.  
  원본에 대한 데이터는 변경을 할 수 없어 안전하지만, 값을 넘길때마다 새로운 메모리 공간을 할당시키기 때문에 메모리 낭비가 발생한다.

</br>

---

- ## **_정리_**

  위에서 주소 값을 참조하여 객체의 프로퍼티에 접근하는 것을 통해 JAVA가 Call By Reference 처럼 동작한다고 착각할 수 있지만, 원본의 데이터의 변경 여부, 값을 넘길 때 복사본을 만드냐 아니면 참조형식이냐를 기준으로 살펴보면 이해할 수 있다.

  자세히 설명하면 Call By Reference에서의 코드는 number의 주소는 100번지라 가정하면, 원본 데이터는 10이라는 값이다. 그리고 함수에 100번지라는 주소값을 줌으로써, 해당 n이라는 포인터 변수 또한 number가 가지고 있는 원본 데이터를 20이라는 값으로 수정을 한 것이다.(Call By Reference)

  하지만 Call By Value에서 main 내에서의 cat1의 주소값은 100번지라 가정하고, 원본 데이터(객체의 주소값)의 값을 200번지라는 값으로 저장하고 있다하자.  
  이후 메서드의 매개변수인 cat1에 main의 cat1의 값을 넘기는데 이때 값은 200번지(객체의 주소값)이고, 메모리 공간을 하나 만들어서 해당 200번지 값을 저장해 둔다. 이러면 cat1 둘 다 같은 객체를 가르키는 상태이며, 위 코드에서 cat1에 새로운 객체를 할당하고 값은 300번지라 하면, 메서드 내의 cat1은 그저 복사본이기 때문에 복사본만이 새로운 객체를 가르킬 뿐 main에 cat1은 200번지 그대로 유지한다.  
  이를 통해 JAVA는 오로지 Call By Value 방식으로만 동작이 된다.

</br>

---

# **_참고_**

- https://mangkyu.tistory.com/106 [JAVA Call By Value 설명]
- https://github.com/gyoogle/tech-interview-for-developer/blob/master/Language/%5Bjava%5D%20Call%20by%20value%EC%99%80%20Call%20by%20reference.md [Call By Value VS Call By Reference]
- https://deveric.tistory.com/92 [Call By Value와 Call By Reference]
