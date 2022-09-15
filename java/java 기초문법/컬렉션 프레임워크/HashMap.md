# **_HashMap_**

HashMap과 HashTable은 ArrayList와 Vector와의 관계와 같다.  
즉, HashTable의 기능을 개선해서 등장한 것이 HashMap이다.  
HashMap의 특징은 Key와 Value 형식으로 데이터를 저장하며, 이때 해싱기법을 통하여 저장한다. 그렇기에 많은 양의 데이터를 검색하는데 최고의 성능을 낸다.

</br>

---

## **_해싱?_**

HashSet, HashMap 의 이름만 봐도 앞에 Hash가 붙는다. 즉, Hash방식으로 데이터를 관리하는 것이다.

그럼 Hash는 무엇인가?

Hash는 **_"해시 함수에 의해 얻어지는 값"_** 을 의미한다.

그럼 해시 함수는 또 무엇인가?

해시 함수는 내부 코드에 의해 임의의 길이를 가진 임의의 데이터를 반환하는 함수이다. 대표적인 해시 함수에는 SHA-256, MD5, SHA-512 등이 존재한다.

그럼 이를 통해 어떻게 HashMap이 key value 형식으로 데이터를 저장하는가?

  <p align = "center"><img src="https://user-images.githubusercontent.com/62879192/189293107-8b29d00f-33bf-4001-aa9a-3c642f00f774.png" height = 250px></p>
  출처) https://ko.wikipedia.org/wiki/%ED%95%B4%EC%8B%9C_%ED%85%8C%EC%9D%B4%EB%B8%94

<p align = "center"><img src="https://user-images.githubusercontent.com/62879192/189293149-8f7e8859-c593-4ed2-83e5-293c34cb5f38.png" height = 250px></p>
출처) https://www.onlybook.co.kr/entry/algorithm-interview

</br>

위처럼 key가 주어지면 해시 함수를 통하여 해시 값을 얻어오고 이를 해시버킷(배열구조)에 해시 값으로 인덱스로 지정하여 해당 인덱스에 value를 넣는다.

그런데 서로다른 key값 (서로다른 객체)인데 같은 해시 값을 반환하여 "해시 충돌" 이라는 문제가 발생한다. 위처럼 "윤아"와 "서현"이 같은 해시를 반환하게 되는데, 이럴때 대표적으로 해결하는 방법은 2가지이다.

위 이미지처럼 value를 LinkedList방식으로 엮어서 해결하는 방식을 "Separate Chaining(개별 체이닝)" 기법이라 한다.

</br>

---

<p align = "center"><img src="https://user-images.githubusercontent.com/62879192/189293163-7f474d78-86bf-40b1-be8e-bd6b7f9bbc34.png" height = 250px></p>

반대로 위 이미지처럼 충돌이 일어날 때 다른 빈 공간을 찾아서 저장하는 방식을 Open Addressing 방식이다.

자바는 개별 체이닝 기법을 택하여 동작이 된다.

자세한 내용은

https://siahn95.tistory.com/entry/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%ED%95%B4%EC%8B%9CHash%EB%9E%80-2-%ED%95%B4%EC%8B%9C-%ED%85%8C%EC%9D%B4%EB%B8%94%EC%97%90%EC%84%9C-%EC%B6%A9%EB%8F%8C-%EC%B2%98%EB%A6%AC-%EB%B0%A9%EC%8B%9D?category=884386

</br>

HashMap은 위처럼 데이터가 저장이 되며, unique한 key값과 value가 1대1로 매핑이 되기 때문에 검색에 매우 효율적이다.

---

## **_HashMap vs HashSet_**

https://siahn95.tistory.com/entry/Java%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-HashMap-HashSet-%EC%9D%B4%EB%9E%80-5-HashMap-vs-HashSet#--%--%EB%-D%B-%EC%-D%B-%ED%--%B-%--%EC%A-%--%EC%-E%A-%--%ED%--%--%ED%--%-C

5개로 나누어서 설명이 잘 되어 있다.
