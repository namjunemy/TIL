> 인프런 - 부경대IT융합응용공학과 궘오흠 교수님의 '**영리한 프로그래밍을 위한 알고리즘 강좌** '([링크](https://www.inflearn.com/course/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B0%95%EC%A2%8C/))와 '**쉽게 배우는 알고리즘** 관계 중심의 사고법 - 문병로' 참조

# 4-1. 트리와 이진트리

## 트리(Tree)

* 계층적인 구조를 표현하기 위해 사용하는 자료구조
  * 조직도
  * 디렉토리와 서브디렉토리 구조
  * 가계도

### 용어

* 루트(Root)
  * 트리는 노드(node)들과 노드들을 연결하는 링크(link)들로 구성된다.
  * 맨 위의 노드를 루트라고 한다.
* 부모-자식(parent-child) 관계
  *  각 노드들의 상하 관계를 부모-자식(parent-child)관계로 나타낸다.
* 형제 관계(sibling)
  * 루트노드를 제외한 트리의 모든 노드들은 유일한 부모노드를 가진다.
  * 부모가 동일한 노드들을 형제 관계라고 부른다.
* 리프(leaf) 노드
  * 자식이 없는 노드들을 leaf노드라고 부른다.
  * 리프노드가 아닌 노드들을 내부(internal)노드라고 부른다.
* 조상-자손(ancestor-descendant) 관계
  * 부모-자식 관계를 확장한 것이 조상-자손 관계이다.
* subtree
  * 트리의 특성상 한 노드의 자손노드들의 집합도 트리이다.
  * 트리에서 어떤 한 노드와 그 노드의 자손들로 이루어진 트리를 subtree라고 부른다.
* 레벨(level)
  * root노드는 level 1
  * root노드의 자식노드들은 level 2
  * ...
* 높이
  * 서로다른 레벨의 수이다.

### 트리의 기본적인 성질

* 노드가 N개인 트리는 항상 N-1개의 링크(link)를 가진다.
* 트리의 루트에서 어떤 노드로 가는 경로는 유일하다. 또한 임의의 두 노드간의 경로도 유일하다. (같은 노드를 두 번 이상 방문하지 않는다는 조건하에)


## 이진 트리(binary tree)

* 이진 트리에서 각 노드는 최대 2개의 자식을 가진다.
* 각각의 자식 노드는 자신이 부모의 왼쪽 자식인지 오른쪽 자식인지가 지정된다. (자식이 한 명인 경우에도)

### 이진 트리 응용의 예: Expression Tree

* 수식을 트리로 구성하여 해석한다.
* 연산자들이 이진연산자라면 이진 트리의 형태로 표현할 수 있다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/bst_01.png?raw=true)

###이진 트리 응용의 예: Huffman Code

* 파일압축과 관련된 유명한 알고리즘 중 하나인 허프만 코딩을 할 때, 각 문자를 encoding하는데 허프만 tree를 구성한다.
* 추후에, 자세히 살펴본다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/bst_02.png?raw=true)

### Full and Complete Binary Trees

* 꽉찬 이진트리와 완전이진트리
* 높이가 h인 full binary tree는 2^h-1개의 노드를 가진다.
* 노드가 N개인 full 혹은 complete binary 트리의 높이는 O(logN)이다. (노드가 N개인 이진트리의 높이는 최악의 경우 N이 될 수도 있다.)


  

## 이진트리의 표현

* 힙을 표현했을 때는, 완전 이진 트리의 형태 였기 때문에 부모와 자식간의 관계가 배열로 표현하기에 충분했다. 하지만, 완전 이진트리가 아닌 형태라면 아래와 같은 구조의 표현이 필요하다.


* 연결구조(Linked Structure)

  * 각 노드에 하나의 데이터 필드와 왼쪽자식(left), 오른쪽 자식(right), 그리고 부모노드(p)의 주소를 저장(부모노드의 주소는 반드시 필요한 경우가 아니면 보통 생략함)

  ![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/bst_03.png?raw=true)

  * 예시
    * 트리상의 노드를 객체로 표현하여 이진트리를 Linked Structure로 표현할 수 있다.

  ![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/bst_04.png?raw=true)

  ​  


## 이진트리의 순회(traversal)

* 순회 
  * 이진 트리의 모든 노드를 방문하는 일
* 중위(inorder) 순회
* 전위(preorder) 순회
* 후위(postorder)순회
* 레벨오더(level-order) 순회

### inorder-중위 순회

* 순회는 본질적으로 recursive한 알고리즘이다.
* 이진트리를 루트노드, left Tree, right Tree로 나누어 생각한다.
* 먼저 left Tree를 inorder로 순회하고,
* root를 순회하고,
* right Tree를 inorder로 순회한다.
* **pseudo code**
  * x를 루트로 하는 트리를 inorder 순회
  * 시간복잡도 O(n)

```java
INORDER-TREE-WALK(x)
  if x != NULL
    then INORDER-TREE-WALK(left[x])
         print key[x]
         INORDER-TREE-WALK(right[x])
```

### postorder와 preorder순회

* inorder 순회와 다른점은 노드를 방문하는 순서뿐이다. 나머지의 개념은 동일하다.

* 트리를 방문하는 순서만 바뀐다.

* preorder-전위 순회

  * root를 순회하고,
  * 먼저 left Tree를 inorder로 순회하고,
  * right Tree를 inorder로 순회한다.

  ```java
  PREORDER-TREE-WALK(x)
    if x != NULL
      then print key[x]
           PREORDER-TREE-WALK(left[x])
           PREORDER-TREE-WALK(right[x])
  ```

* postorder-후위 순회

  * 먼저 left Tree를 inorder로 순회하고,
  * right Tree를 inorder로 순회하고,
  * root를 순회한다.

  ```java
  POSTORDER-TREE-WALK(x)
    if x != NULL
      then POSTORDER-TREE-WALK(left[x])
           POSTORDER-TREE-WALK(right[x])
           print key[x]
  ```

### Expression Trees

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/bst_05.png?raw=true)

* Expression 트리를 inorder-중위 순회하면 다음과 같이 출력됨
  * x + y * a + b / c
* 각 부트리를 순회할 때 시작과 종료시에 괄호를 추가하면 다음과 같이 올바른 수식이 출력됨
  * (x + y) * ((a + b) /c)
* postorder-후위 순회하면 후위표기식이 출력됨
  * x y + a b + c / *

### lever-order 순회

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/bst_06.png?raw=true)

* 레벌 순으로 방문, 동일 레벨에서는 왼쪽에서 오른쪽 순서로 방문
* 큐(queue)를 이용하여 구현
  * 맨 처음 큐에 루트 노드 3을 넣는다.
  * 큐를 꺼내고, 3을 출력한 뒤,
  * 3의 왼쪽자식 1과, 오른쪽자식 5를 큐에 넣는다.
  * 1을 꺼내고 출력한 뒤, 1의 자식인 0과 2를 큐에 넣는다.
  * 다음 순서로 5를 꺼내고 출력한 뒤, 5의 왼쪽 자식인 4와 오른쪽 자식인 6을 큐에 넣는다.
* pseudo code

```java
LEVEL-ORDER-TREE-TRAVERSAL()
  visit the root;
  Q <- root       //Q is a queue
  while Q is not empty do
    v <- dequeue(Q);
    visit children of v;
    enqueue children of v into Q;
  end.
end.
```


