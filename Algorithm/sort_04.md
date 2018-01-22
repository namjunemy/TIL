# 3-4. 힙 정렬(Heap Sort) 1

## Heap과 Heap sort

* 최악의 경우 시간복잡도 O(nlogn)
* Sorts in place - 추가 배열 불필요
  * mergesort도 최악의경우 O(nlogn)이었지만, 추가 배열이 필요했음.
* 이진 힙(binary heap) 자료구조를 사용
* O(nlogn)의 시간복잡도를 가지면서, merge sort처럼 추가적인 배열이 필요하지 않기 때문에 좋은 정렬 알고리즘 중 하나다.

## Heap의 정의

* Heap은
  * 완전 이진 트리(complete binary tree)이면서
  * Heap property를 만족해야 한다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/heap_02.png?raw=true)

* 동일한 데이터를 가진 서로 다른 힙이 존재할 수 있다. 즉, 힙은 유일하지 않다.(같은 원소들을 가지는는데 다른 위치에 가진다.)
* 힙은 일차원 배열로 표현가능하다. A[1...n]
  * 루트 노드 : A[1]
  * A[i]의 부모 : A[i/2]
  * A[i]의 왼쪽 자식 = A[2i]
  * A[i]의 오른쪽 자식 = A[2i + 1]

  

### Full vs Complete Binary Trees

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/heap_01.png?raw=true)

  

## 기본 연산: Max-Heapify

* 전체를 힙으로 만들어라.
  * 왼쪽 자식 트리와 오른쪽 자식 트리가 모두 heap property를 만족하는데, root만이 조건을 만족하지 않을 때, 이 트리를 힙으로 만드는 연산을 heapify라 한다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/heap_03.png?raw=true)

* max-heapify 연산은 아래와 같은 상황전개가 이루어진다.
  * 두 자식노드 중에 큰 값을 선택하여 교환한다.
  * 4와 16을 교환하고 나면, 오른쪽 트리가 heap property를 만족하는지 고민할 필요가 없다. 16과 15를 비교하여 큰 노드와 교환했기 때문에, 오른쪽 트리는 조건을 만족한다.
  * 4와 16을 교환하고 나면, 4를 루트노드로 봤을 때, 그 아래의 트리들은 같은 상황을 맞게 된다.
  * 왼쪽, 오른쪽 자식 힙이 property를 만족하는데, 루트노드인 4만 조건을 만족하지 않는 상황이다.
  * 같은 방법으로 두 자식중에 큰 값인 8과 루트인 4를 교환한다.
  * 그러고나면 오른쪽 힙인 7은 heap property를 만족하게 되므로 신경쓸 필요가 없으며,
  * 이러한 방식으로 루트노드인 4가 더이상 비교할 자식이 없거나, 두 자식들이 루트 노드 보다 작다면(루트 노드가 들어갈 자리를 찾았다면) 종료한다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/heap_04.png?raw=true) 

  

## Max-Heapify: recursion version

* heapify는 본질적으로 recursive한 특성을 가지고 있다.
  * 교환이 일어나고 나면, 반대쪽 힙은 생각하지 않고 반복적으로 교환이 일어난 쪽의 자식 힙만 생각하면 된다.
* root 노드에 대한 heapify는 maxHeapify(1)을 호출하면 되고, 루트노드는 i
* base case는 i의 자식이 없는 경우
* i는 heapify의 대상노드 즉, 시작노드
* k는 i의 자식노드 중 큰 쪽

```java
maxHeapify(A, i) {
  if there is no child of A[i]
    return;
  k <- index of the biggest child of i;
  if A[i] >= A[k]
    return;
  exchange A[i] and A[k];
  maxHeapify(A, k);
}
```

  

## Max-Heapify: iterative version

* i=k;

```java
maxHeapify(A, i) {
  while A[i] has a child do
    k <- index of the biggest child of i;
    if A[i] >= A[k]
      return;
    exchange A[i] and A[k];
    i=k;
  end.
}
```

  

## Heapify연산의 시간복잡도

* 두 자식들 중 더 큰 쪽을 찾아서 exchange하는 연산을 하면, Heap의 Tree에서 한 레벨 내려온다.
* 그러므로, Heapify 알고리즘은 어떠한 경우에도 트리의 높이보다 더 많은 시간이 필요하지 않는다.
* 따라서, 트리의 높이를 h라고 하면 시간복잡도는 O(h)가 된다.
* 여기서 h를 구해보자.
* heap은 complete binary tree이기 때문에 노드의 수를 n이라고 했을 때, h는 Θ(logn)이 된다.
* 따라서, Θ(logn)이며 n은 노드의 갯수이다.

