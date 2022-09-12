# **_LinkedList_**

배열(Array)은 가장 기본적인 자료구조로, 쉽게 사용할 수 있으며 데이터를 읽어 오는 시간이 매우 짧다는게 장점이다.

단점은 배열의 크기를 변경할 수 없다는 것이다.  
배열의 크기보다 더 큰 데이터를 저장하기 위해서는 새로운 공간을 할당을 받아야 되기 때문에 메모리 공간이 낭비가 된다.

다음은 비순차적인 데이터의 추가/삭제에 대한 내용이다.  
배열의 경우 순차적으로 추가/삭제하는 것은 빠르게 처리가 가능하지만, 중간에 추가/삭제 작업을 하는 경우 데이터를 복사해서 작업을 해야하기 때문에 많은 시간이 소요가 된다.

이러한 배열의 단점을 해결한 것이 **_LinkedList_** 이다.

```java
package chapter11;

import java.util.LinkedList;

public class LinkedListEx {
    public static void main(String[] args) {
        Node node1 = new Node(10);
        Node node2 = new Node(new Integer(20));
        Node node3 = new Node(30);

        node1.node = node2;
        node2.node = node3;

        Node node = node1;

        System.out.println(node.node == node2); //true
        nodeTest(node);
//        System.out.println(node.node.node.node.obj); NPE 발생

        //node 추가
        Node node4 = new Node(15);
        node4.node = node1.node; //node2의 주소
        node1.node = node4; //node4의 주소

        nodeTest(node);
    }

    public static void nodeTest(Node node){
        System.out.println();
        while(node != null){
            System.out.println(node.obj);
            node = node.node;
        }
    }
}

class Node {
    Node node;
    Object obj;

    public Node(Object obj){
        this.obj = obj;
    }
}

실행결과 :
true

10
20
30

10
15
20
30
```

LinkedList를 코드로 구현한 방식이며, 위와 같이 주소들끼리 연결이 되어 있어 불규칙적인 주소값이어도 서로 접근이 가능하다.

양방향으로 주소를 참조해서 양쪽으로 접근이 가능하게 구현하면, 이중 연결 리스트  
맨처음과 맨끝의 주소를 연결시키면 원형 연결 리스트가 되는 것이다.

```java
        LinkedList list = new LinkedList();
        list.add("가");
        list.add("다");
        list.add(1,"나");

        System.out.println(list);
        list.remove(1);

        System.out.println(list);

실행결과 :
[가, 나, 다]
[가, 다]
```

위와 같이 자바 내에 LinkedList 컬렉션 클래스가 이미 존재하며, 여러 메서드들이 내장되어 있다.

</br>

---

## **_ArrayList VS LinkedList_**

LinkedList가 배열의 단점들을 해결했다고해서 완벽한 것은 아니다.

LinkedList는 추가/삭제 시에 그저 주소값만 조금만 변경하면 되기 때문에 중간에 데이터를 추가/삭제 시에는 배열보다 효율이 좋다. 하지만 맨 끝에서 추가/삭제만 이루어지는 경우(stack)에는 LinkedList보단 배열이 효율적일 것이다.

그리고 LinkedList의 문제점은 n번째 데이터를 읽어 오는 과정에서 생긴다.

배열의 경우 배열주소 + n \* 데이터 타입 크기 를 통하여 손쉽게 데이터를 읽을 수 있지만, LinkedList의 경우에는 n번째 데이터를 찾기 위해서는 처음 주소부터 차례대로 접근해야 하기 때문에 데이터 읽는시간이 많이 소요가 된다.

그래서 데이터가 변하지 않거나, 맨 뒤에서만 추가/삭제가 이루어지는 경우에는 ArrayList를 사용하고 데이터의 길이가 길지 않거나 데이터를 자주 읽지는 않고, 데이터의 개수 변경이 잦을 때에는 LinkedList가 효율적일 수 있다.

</br>

---

## **_Stack과 Queue_**

추가로, Stack과 같이 맨 끝에서만 추가/삭제가 이루어지는 LIFO 구조에서는 ArrayLIst가 좋고 Queue와 같이 FIFO 구조에서는 ArrayList로 구현할 경우 데이터의 추가/삭제가 이러날때마다 데이터를 복사해서 이동시켜야 하므로, LinkedList가 효율적이다.
