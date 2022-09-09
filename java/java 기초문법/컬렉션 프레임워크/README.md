# **_컬렉션 프레임워크_**

## **_컬렉션 프레임워크란?_**

많은 데이터를 쉽고 효과적으로 처리할 수 있도록 도와주는 클래스들의 집합

</br>

---

## **_종류_**

<p align ="center"><img src="https://user-images.githubusercontent.com/62879192/188573845-e8252ea0-79fb-4156-a980-1676383a9b6c.png"></p>

List 와 Set 의 두 인터페이스는 공통된 부분이 많이 존재하여, 두 인터페이스의 공통점을 Collection 이라는 인터페이스에 정의하여 상속받게 구현하였다.

Map은 List와 Set과는 성질이 달라서 따로 존재한다.

List의 특징으로는 데이터의 순서의 보장과 데이터 중복의 허용이며, List 인터페이스를 상속받는 클래스들은 ArrayList, LinkedList, Vector, Stack이 존재한다.

Set의 특징으로는 데이터의 순서 보장이 되지않고, 중복 또한 허용이 되지 않는다.  
상속받는 클래스들은 HashSet 그리고 SortedSet에서 상속받아서 사용하는 TreeSet이 존재한다.

Map의 특징으로는 Key, Value 형태로 데이터를 저장하며 데이터 순서 보장이 되지 않고, Key값은 중복이 안되지만 Value(값)은 중복 허용이 된다.  
상속받는 클래스들은 Hashtable, HashMap, SortedMap에서 부터의 TreeMap 이 존재한다.

</br>

---

# **_참고_**

https://gangnam-americano.tistory.com/41
