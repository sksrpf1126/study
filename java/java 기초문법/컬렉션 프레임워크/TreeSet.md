# **_TreeSet_**

TreeSet은 이진 검색 트리의 형식으로 데이터를 저장한다.  
그렇기에 자동으로 정렬이 되기 때문에 정렬, 검색, 범위 검색에서의 성능이 좋다.  
이진 검색 트리 형식으로 데이터를 저장하지만 구현된 방식은 Red-black 트리 방식이다. 이유는 이진 검색 트리에서 계속 큰 수가 올 경우에는 한쪽으로 편향된 트리가 나타나면서, 트리의 높이가 높아져 탐색 시간이 길어진다. 그래서 이러한 문제와 이진 검색 트리에서 성능을 향상시킨 Red-Black 트리 방식으로 구현한 것이다.

```java
public class TreeSetEx1 {
    public static void main(String[] args) {
        //7,4,9,1,5 순으로 데이터를 넣는 경우
        TreeNode rootNode = new TreeNode(7);
        rootNode.nodeInsert(new TreeNode(4));
        rootNode.nodeInsert(new TreeNode(9));
        rootNode.nodeInsert(new TreeNode(1));
        rootNode.nodeInsert(new TreeNode(5));

        /**
         * 예상 구조
         *                                                       7(rootNode.value)
         *                          4(rootNode.leftNode.value)                     9(rootNode.rightNode.value)
         *   1(rootNode.leftNode.leftNode.value)             5(rootNode.leftNode.rightNode.value)
         */

        System.out.println(rootNode.value); //7
        System.out.println(rootNode.leftNode.value); //4
        System.out.println(rootNode.rightNode.value); //9
        System.out.println(rootNode.leftNode.leftNode.value); //1
        System.out.println(rootNode.leftNode.rightNode.value); //5

        System.out.println();

        //중위순회 구현
        TreeNode.inOrderView(rootNode);

    }
}

class TreeNode {
    TreeNode leftNode;
    int value;
    TreeNode rightNode;

    //rootNode를 받아 중위순회를 통해 오름차순 정렬
    public static void inOrderView(TreeNode node){
        if(node == null) return;

        if(node.leftNode != null){
            inOrderView(node.leftNode);
        }

        System.out.print(node.value + "  "); //값 출력

        if(node.rightNode != null){
            inOrderView(node.rightNode);
        }
    }

    public TreeNode(int value){
        this.value = value;
    }

    //현 노드보다 작은 값은 왼쪽, 큰 값은 오른쪽으로 노드 저장되게 간단하게 작성
    public void nodeInsert(TreeNode node){
        if(node == null || this.getClass() != node.getClass()) return;

        if(this.value > node.value){
            if(this.leftNode == null){
                this.leftNode = node;
            }else{
                this.leftNode.nodeInsert(node);
            }
        }else if(this.value == node.value){
            System.out.println("이미 값이 존재합니다.");
            return;
        }else{
            if(this.rightNode == null){
                this.rightNode = node;
            }else{
                this.rightNode.nodeInsert(node);
            }
        }
    }
}

실행결과 :
7
4
9
1
5

1  4  5  7  9
```

위는 이진 검색 트리의 동장 방식을 자바코드로 구현해 본 것이다.  
중위 순회 방식으로 트리를 조회하면 오름차순으로 정렬이 된다.

</br>

```java
public class TreeSetEx2 {
    public static void main(String[] args) {
        Set treeSet = new TreeSet();

        for(int i = 0; treeSet.size() < 6; i++){
            int num = (int)(Math.random() * 45) + 1;
            treeSet.add(num);
        }

        //TreeSet은 알아서 정렬을 해줌
        System.out.println(treeSet);
    }
}

실행결과 :
[4, 20, 22, 29, 34, 43]
```

중복을 허용하지 않기 때문에 6개의 중복없는 수가 뽑힐 때까지 반복문을 실행하며, TreeSet은 정렬을 자동으로 해주기 때문에 바로 출력하면 정렬된 결과가 나온다.

</br>

```java
public class TreeSetEx3 {
    public static void main(String[] args) {
        TreeSet ts = new TreeSet();

        String from = "b";
        String to = "d";

        ts.add("abc"); ts.add("alien"); ts.add("bat");
        ts.add("car"); ts.add("Car"); ts.add("disc");
        ts.add("dance"); ts.add("dZZZZ"); ts.add("dzzzz");
        ts.add("elephant"); ts.add("elevator"); ts.add("fan");
        ts.add("flower");

        System.out.println(ts);
        System.out.println("range Search : from " + from + " to " + to);
        System.out.println("result1 : " + ts.subSet(from, to));
        System.out.println("result1 : " + ts.subSet(from, to + "zzz"));

    }
}

실행결과 :
[Car, abc, alien, bat, car, dZZZZ, dance, disc, dzzzz, elephant, elevator, fan, flower]
range Search : from b to d
result1 : [bat, car]
result1 : [bat, car, dZZZZ, dance, disc]

```

TreeSet은 위와같이 범위검색에도 효율이 뛰어나며, to는 포함하지 않기 때문에 d로시작하는 문자열은
출력이 되지 않는다.  
그래서 위와 같이 to + "zzz" 를 함으로써 dzzz 안의 범위까지 조회하면 dzzz안의 범위까지
조회가 된다.  
dzzz는 안나오는데 dZZZ가 나오는 이유는 소문자보다 대문자를 우선시 하기 때문이다.

TreeSet은 추가/삭제가 LinkedList보다 수행시간이 길다. 그 이유는 정렬에 의해 데이터가 어디에 들어가야할지 정해진 후 추가가 이루어지고, 삭제 또한 중간에 삭제가 일어나면 삭제 후 일부 트리 구조를 변경해야 하는 문제가 생기기 때문이다.

하지만 배열이나 LinkedList에 비해 검색과 정렬기능이 뛰어나다.

TreeSet에 POJO를 저장할 때에는 그에 맞는 Comparable 또는 Comparator를 통해 객체간 비교하는 방법을 정의해줘야 한다. 아니면 예외가 발생.
