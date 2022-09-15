# **_Iterator_**

Iterator의 구버전은 Enumeration 이며, Iterator의 양방향 조회기능이 추가된 것이 ListIterator(List를 구현한 경우에만 가능)이다.

3개 모두 인터페이스이며, Iterator를 중점으로 설명

Iterator는 컬렉션에 저장된 요소들에 접근하기 위해 사용되는 인터페이스들이다.

```java
//        ArrayList list = new ArrayList();
        LinkedList list = new LinkedList();
        list.add(1);
        list.add(2);
        list.add(3);

        Iterator i = list.iterator();

        /**
         * 다른 컬렉션 클래스로 변경하여도 동일하게 요소에 접근 가능
         * 코드의 재사용성이 향상
         */
        while(i.hasNext()){
            System.out.println(i.next());
        }

        while(i.hasNext()){
            System.out.println("실행 X");
            System.out.println(i.next());
        }

실행결과 :
1
2
3
```

Iterator는 위처럼 다른 컬렉션 클래스로 변경하여도 해당 인터페이스에 대한 인스턴스만 얻는다면 코드의 변경없이 요소에 접근할 수 있다. 그렇기에 코드의 재사용성이 향상이 되는 것이다.

</br>

---

## **_MAP과 Iterator_**

Map인터페이스를 구현한 컬렉션 클래스들은 바로 Iterator 인스턴스를 얻지는 못하고, keySet() 또는 entrySet()을 통해 각각 Set 형태로 얻어 온 후에 Iterator 인스턴스를 얻어와야 한다.

```java
Map map = new HashMap();
Iterator it = map.entrySet().iterator();
```
