# **_B-tree, B+tree_**

대부분의 DBMS에서 인덱스는 B+tree 알고리즘을 활용한다. 그리고 B+tree는 B-tree에서 발전(?)된 알고리즘이기 때문에 해당 내용에서는 B-tree와 B+tree가 무엇인지, 차이점은 무엇인지 설명한다.

</br>

---

## **_B-tree_**

B트리(B-tree)는 이진트리에서 발전되어 모든 리프노드들이 같은 레벨을 가질 수 있도록 자동으로 밸런스를 맞추는 트리이다. 노드의 내부의 값으로는 여러개의 키값을 가질 수 있으며, 키는 data와 매핑이 된다.  
또한, 키 값들 간에는 정렬된 값이 보장이 된다.  
이러한 특징을 통해 DB는 효율적으로 데이터를 탐색할 수 있다.

또한, 탐색할 때 루트노드에서 어떠한 리프노드들에 접근한다 하여도 같은 깊이로 탐색을 하기 때문에 DB의 입장에서는 균등한 검색속도를 낼 수 있다.

아래 내용은 블로그에서 참고한 글이며, B트리의 특성을 나열한 것이다.

```
B트리는 이진트리와 다르게 하나의 노드에 많은 수의 정보를 가지고 있을 수 있습니다. 최대 M개의 자식을 가질 수 있는 B트리를 M차 B트리라고 하며 다음과 같은 특징을 같습니다.

노드는 최대 M개 부터 M/2개 까지의 자식을 가질 수 있습니다.
노드에는 최대 M−1개 부터 [M/2]−1개의 키가 포함될 수 있습니다.
노드의 키가 x개라면 자식의 수는 x+1개 입니다.
최소차수는 자식수의 하한값을 의미하며, 최소차수가 t라면 M=2t−1을 만족합니다. (최소차수 t가 2라면 3차 B트리이며, key의 하한은 1개입니다.)

```

</br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/210998557-0a1d41bf-8dda-47ba-84b5-b9da45f1cf59.png" width = 80%>
  </p>

출처 : https://velog.io/@emplam27/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EB%8A%94-B-Tree

위는 B-tree의 구조를 나타낸다.

부모노드의 내부 키값들 중에 제일 왼쪽의 값보다 작은 값들은 왼쪽 서브트리로 이동하며, 제일 오른쪽 값보다 클 경우 오른쪽 서브트리로 이동하는 형식으로 탐색한다. 그 사이의 값이라면 가운데 서브트리로 이동한다.

예를 들어 11이라는 값을 탐색하고자 한다면 11은 10과 20 사이의 값이므로, 가운 데 서브트리로 이동하게 된다. 이후, 14와 비교했을때 값이 더 작으므로 왼쪽으로 이동하게 된다.  
그러면 11과 12의 값을 가진 리프노드에서 11의 값을 찾을 수 있게 되는 것이다.

Key값을 삽입하거나 삭제했을 때는 여러 경우의 수가 존재하므로 아래에 매우 자세히 설명이 되어있다.  
https://velog.io/@emplam27/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EB%8A%94-B-Tree

</br>

---

## **_B+ tree_**

B+ tree가 B트리와 다른점은 B트리는 모든 노드들에서 Key에 대한 data를 가지지만, B+ tree는 리프노드들에서만 Key에 대한 data를 가진다.  
또한, 리프노드들간에 연결리스트의 형태로 연결이 되어 있다.

</br>

  <p align = "center">
  <img src="https://user-images.githubusercontent.com/62879192/211000179-f3a8a889-db88-4a54-85e2-b842fb3631c2.png" width = 80%>
  </p>

출처 : https://velog.io/@emplam27/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EB%8A%94-B-Plus-Tree

위 구조가 B+ tree 구조이며, 보통의 DB들은 인덱스를 구성할 때 위의 B+ tree 알고리즘을 사용한다.

리프노드들간에 연결리스트로 연결이 되어 있기 때문에 리프노드에서 선형검사(연결리스트를 통한 데이터 탐색)이 가능해진다. 그렇기에 시간복잡도가 줄어든다.  
예를 들어 B트리의 경우에는 리프노드를 탐색하고나서, 바로 옆의 리프노드를 찾고자 한다면 다시 루트노드에서부터 탐색을 해야하지만, B+ tree는 그럴필요 없이 연결리스트 방식으로 탐색하면 되기 때문에 성능이 더 좋다는 것이다.

B+ tree는 탐색에 대해선 B트리와 동일하지만 삽입과 삭제에 대해서는 다르게 동작하므로 잘 정리된 아래의 글을 참고하자.

https://velog.io/@emplam27/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EB%8A%94-B-Plus-Tree

</br>

---

# **_참고_**

https://velog.io/@emplam27/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EB%8A%94-B-Tree [B-tree]  
https://velog.io/@emplam27/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EB%8A%94-B-Plus-Tree [B+tree]  
https://kyungyeon.dev/posts/66 [B-tree를 통해 알아보는 Index]
