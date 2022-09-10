# **_ArrayList_**

ArrayList 는 컬렉션 프레임워크에서 가장 많이 사용되는 클래스들 중 하나이다.  
데이터의 순서 보장, 값 중복 허용의 특징이 존재하며, ArrayList는 Vector 클래스를 개선한 것으로, Vector클래스는 기존에 작성된 소스코드와의 호환성 때문에 남겨두고 있을 뿐이므로, Vector를 쓸 상황이면 ArrayList로 대체해서 사용하자.

ArrayList는 내부적으로 Object[] 즉, 객체 배열 형식으로 값을 담는다. 그렇기에 기본형으로 넣을 경우에는 오토박싱에 의하여 객체로 저장이 되는듯하다.

</br>

---

## **_ArrayList 특징_**

```java
public static void main(String[] args) {
    ArrayList list1 = new ArrayList(10);
    list1.add(5); //new Integer(5);
    list1.add(4);
    list1.add(2);
    list1.add(0);
    list1.add(1);
    list1.add(3);

    ArrayList list2 = new ArrayList(list1.subList(1,4)); //인덱스1 ~ 3까지 자른다.
    print(list1, list2);

    Collections.sort(list1);
    Collections.sort(list2);
    print(list1,list2);

    list2.add("B");
    list2.add("C");
    list2.add(3, "A"); //중간 인덱스에 추가 가능 (기존 인덱스 3의 앞에)
    print(list1, list2);

    list2.set(3,"AA"); //특정 인덱스의 값 변경
    print(list1, list2);

    //list1을 list2와 겹치는 부분만 남기고 나머지는 삭제(인덱스 순서는 상관 X, 값을 기준)
    System.out.println("list1.retainAll(list2) : " + list1.retainAll(list2));
    print(list1, list2);

    //list1이 가지고 있는 객체가 list2에 존재한다면 list2에서 해당 객체 제거
    for(int i= list2.size() -1; i >=0; i--){
        if(list1.contains(list2.get(i))){
            list2.remove(i);
        }
    }

    print(list1, list2);
}

public static void print(ArrayList list1, ArrayList list2){
    System.out.println("list1 : " + list1);
    System.out.println("list2 : " + list2);
    System.out.println();
}

실행 결과 :
list1 : [5, 4, 2, 0, 1, 3]
list2 : [4, 2, 0]

list1 : [0, 1, 2, 3, 4, 5]
list2 : [0, 2, 4]

list1 : [0, 1, 2, 3, 4, 5]
list2 : [0, 2, 4, A, B, C]

list1 : [0, 1, 2, 3, 4, 5]
list2 : [0, 2, 4, AA, B, C]

list1.retainAll(list2) : true
list1 : [0, 2, 4]
list2 : [0, 2, 4, AA, B, C]

list1 : [0, 2, 4]
list2 : [AA, B, C]
```

ArrayList에서 제공하는 메서드들을 통해 작성한 예제들이다.

ArrayList의 특징으로는

첫번째 : 더 이상 저장할 공간이 없을 경우에는 해당 배열의 크기를 옆으로 늘리는것이 아니라 기존의 배열 크기보다 더 큰 배열을 만들어서 값을 복사하여 저장시킨다. 이후 해당 배열을 사용하는 것이다.

두번째 : ArrayList에서 추가와 삭제의 동작 방식이다.  
ArrayList는 특정 인덱스를 삭제할 때에 해당 인덱스가 마지막 인덱스가 아닌 경우에는 특정 인덱스 아래의 모든 인덱스의 값들을 복사하여 해당 인덱스로 한칸 올려서 복사하여 덮어씌우는 방식으로 제거한다.  
마지막 인덱스라면 그저 값만 null로 변경하면 되지만, 중간에 추가 또는 삭제하는 경우에는 다른 데이터들이 전부 이동이 되어야 하기 때문에 데이터가 커지면 커질수록 작업 소요시간이 길어진다는 것이다.
