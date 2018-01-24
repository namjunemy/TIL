# 3-5. 우선순위 큐(priority queue)

## 힙의 응용: 우선순위 큐

* 최대 우선순위 큐(maximum priority queue)는 다음의 두 가지 연산을 지원하는 자료구조이다.
  * INSERT(x) : 새로운 원소 x를 삽입
  * EXTRACT_MAX(): 최대값을 삭제하고 반환
* 최소 우선순위 큐(minimum priority queue)는 EXTRACT-MAX대신 EXTRACT-MIN을 지원하는 자료구조
* MAX HEAP을 이용하여 최대 우선순위 큐를 구현

### INSERT

* max heap의 형태로 원소들이 저장되어 있고, 15가 insert되는 경우 아래와 같은 과정을 거친다.
* complete binary tree를 만족하면서, max heap property를 만족하도록 만들어 준다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/priority_queue_01.png?raw=true)

* **pseudo code**
  * Heap의 size를 하나 늘리고, key값을 배열의 마지막 노드에 삽입
  * i는 처리할 문제 노드
  * while문 -> 루트 노드가 아니고(i > 1), 부모노드의 값보다 문제 노드의 값이 더 크면, 서로 교환하고, 문제노드는 PARENT가 된다.
  * 시간복잡도 : O(h), h는 트리의 높이 이므로 시간복잡도는 O(logn). 문제 노드가 비교연산을 거칠 때마다, 위로 한 레벨 올라가거나 또는 루트노드에 도달하면 종료하므로 트리의 높이에 비례한다. Complete binart tree이므로 트리의 노드를 n개라고 했을 때, tree의 높이는 logn이다.

```java
MAX-HEAP-INSERT(A, key) {
  heap_size = heap_size + 1;
  A[heap_size] = key;
  i = heap_size;
  while (i > 1 and A[PARENT(i)] < A[i]) {
    exchange A[i] and A[PARENT(i)];
    i = PARENT(i);
  }
}
```

### EXTRACT_MAX()

* 최대값을 힙으로 부터 삭제하고, 반환해주는 메소드
* max heap에서 최대값은 항상 root에 존재한다.
* 따라서, heap으로 부터 root에 있는 값을 삭제하고 반환한다.
* Complete binary tree에서 노드 하나를 삭제하고, 그것이 다시 complete binary tree를 만족하게 하려면, 마지막 노드를 삭제하는 방법밖에 없다. 그러나 마지막 노드의 데이터를 삭제하면 안된다.
* 결과적으로,
* root노드의 값을 삭제하고 반환한 뒤, 마지막 노드의 값인 6을 root노드로 옮긴다.
* 다음으로 heap property를 만족하지 않는 힙에 대해 다시 max heap을 만드는 Heapify()연산을 수행한다. root노드 부터.
* heapify() 시간복잡도는 O(logn)

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/priority_queue_02.png?raw=true)

* **pseudo code**
  * 1,2 : heap size 예외처리
  * 3 : 최대값을 max에 저장 후 마지막에 return
  * 4 : 배열의 맨 마지막 노드를 루트노드로 카피한다.
  * 5 : 힙의 사이즈를 1줄이면, 마지막 노드를 제거하는 것과 같다.
  * 6 : 루트노드에 대해서 max heapify를 수행한다.
  * 7 : 저장했던 최대 값을 리턴한다.
  * 시간복잡도는 O(logn)

```java
HEAP-EXTRACT-MAX(A)
1  if heap-size[A] < 1
2    then error "heap underflow"
3  max <- A[1]
4  A[1] <- A[heap-size[A]]
5  heap-size[A] <- heap-size[A] - 1
6  MAX-HEAPIFY(A, 1)
7  return max
```

