# 3-4. 힙 정렬(Heap Sort)

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


## 정렬할 배열을 힙으로 만들기

* heap과 heapify연산을 이용하여 정렬된 배열을 만드는 알고리즘에 대해 알아본다.
  * 시간복잡도: O(n)
  * length[A]: 정렬할 데이터의 개수
  * for 문을 length/2 부터 시작하는 이유는 leaf노드가 아닌 첫번째 노드 즉, 리프노드의 부모노드 부터 heapify연산의 가능 여부를 판단하기 때문이다.(아래의 설명 참조)
  * Max-heapify를 실행한다.
  * 완전 이진 트리의 형태(노드의 갯수가 n개)를 가지는 힙의 heapify연산이 O(logn)이다. 이 연산을, n/2번 수행하므로, 시간복잡도는 O(nlogn)으로 볼 수 있는데,
  * 이 경우는 일반적인 경우보다 과도하게 많이 측정한 시간이 된다. 왜냐면, 항상 루트노드에 대해서 heapify를 하는 것이 아니라, 마지막 leaf노드의 부모에서 부터 heapify를 수행하므로 첫번째 heapify의 경우 노드의 갯수가 2개 또는 3개이므로.
  * 좀 더 정확히 분석을 하면 정렬할 배열을 힙으로 만드는 데에는 O(n)이 된다. 증명에 관한 부분은 책을 참고한다.
  * Heap sort에서는 실제로 힙을 정렬하는 과정에서 O(nlogn)의 시간복잡도를 갖기 때문에, 힙을 만드는 과정의 시간복잡도가 O(n)이던, O(nlogn)이던 전체 힙 정렬의 시간복잡도는 O(nlogn)이 된다. 따라서, 힙을 만드는데 필요한 시간복잡도에 주목하지 않아도 된다.

```java
BUILD-MAX-HEAP(A)
  1 heap-size[A] <- length[A]
  2 for i <- length[A]/2 downto 1
  3   do MAX-HEAPIFY(A, i)
```

* 먼저 주어진 1차원 배열을 complete binary tree로 해석한다. 실제로 트리를 만든다는 것이 아니라, 개념적으로 tree로 생각한다는 의미이다. 

* 다음으로 complete binary tree를 heap으로 바꾼다.

  * Level order의 역순으로 노드들을 고려했을 때, leaf노드가 아닌 첫번째 노드(16)부터, 그 노드를 루트 노드로 하는 sub tree에 대해서 heapify연산을 할 수 있는 조건(양쪽 서브트리가 모두 heap인가)을 확인한다. 
  * 다음 순서로 2에 대해 양쪽 sub tree에 대해 양쪽 섭즈트리가 모두 heap인가를 확인한다. 싱글노드이므로 힙이다. 따라서 2는 heapify 연산을 하기 위한 조건이 된다. 따라서, heapify연산을 수행하면 2와 14가 exchange된다.
  * 같은 방식으로 level order의 역순으로 올라가면서, heapify연산을 수행한다.
  * 결과적으로 f 단계와 같이 max heap을 얻을 수 있다.
  * 이것을 pseudo code로 작성하면 위의 코드와 같이 단순하게 작성할 수 있다.

  ![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/heap_05.png?raw=true)

### 실제 입력 배열을 힙으로 만드는 과정

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/heap_06.png?raw=true)

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/heap_07.png?raw=true)

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/heap_08.png?raw=true)



## Heap Sort

* 주어진 데이터로 힙을 만든다.
* 힙에서 최댓값(루트)을 가작 마지막 값과 바꾼다.
* 힙의 크기가 1 줄어든 것으로 간주한다. 즉, 가장 마지막 값은 힙의 일부가 아닌 것으로 간주한다.
* 루트 노드에 대해서 HEAPIFY(1)을 실행한다.
* 2~4번을 반복한다.
* 마지막노드와 루트노드를 바꾸고 힙의 크기를 1줄이면, 루트노드를 제외한 모든 곳에서 heap property를 만족하므로 heapify(1)을 실행해주면 된다.

### pseudo code

* 먼저 배열 A를 max Heap으로 만든다. O(n)의 시간이 든다.
* Heap size가 2가될 때 까지 반복하고,
* 루트노드와 마지막노드를 교환하고, 힙의 사이즈를 1줄인다.
* Max-Heapify를 호출한다.

```java
HEAPSORT(A)
1. BUILD-MAX-HEAP(A)                 : O(n)
2. for i <- heap_size downto 2 do    : n-1 times
3.   exchange A[1] <-> A[i]          : O(1)
4.   heap_size <- heap_size - 1      : O(1)
5.   MAX-HEAPIFY(A, 1)               : O(logn)
```

 