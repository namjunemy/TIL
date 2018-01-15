# 3-1. 정렬

* simple, slow
  * Bubble sort
  * Insertion sort
  * Selection sort
* fast
  * Quick sort
  * Merge sort
  * Heap sort
* O(n)
  * Radix sort

## 기본적인 정렬 알고리즘

### Selection Sort

* 각 루프마다
  * 최대 원소를 찾는다
  * 최대 원소와 맨 오른쪽 원소를 교환한다.
  * 맨 오른쪽 원소를 제외한다.
* 하나의 원소만 남을 때까지 위의 루프를 반복한다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/sort_01.png?raw=true)

* pseudocode

```java
selectionSort(A[], n) {
  for last <- n downto 2 {                              == 1
    A[1,...,last] 중 가장 큰 수 A[k]를 찾는다;               == 2
    A[k] <-> A[last];  // A[k]와 A[last]의 값을 교환       == 3
  }
}
```

* 실행시간
  * 1의 for 루프는 n-1번 반복
  * 2에서 가장 큰 수를 찾기 위한 비교횟수: n-1, n-2, ... , 2, 1
  * 3의 교환은 상수시간 작업
* 시긴복잡도  T(n) = (n-1) + (n-2) + … + 2 + 1 = O(n<sup>2</sup>)
  * 최악, 최선, 평균 항상 n(n-1) / 2번의 비교연산을 수행하게 되므로 O(n<sup>2</sup>)이다.

### Bubble Sort

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/sort_02.png?raw=true)

* 실행시간
  * (n-1) + (n-2) + … + 2 + 1 = O(n<sup>2</sup>)
* pseudocode

```java
bubbleSort(A[], n) {
  for last <- n downto 2 {                    == 1
    for i <- to last-1                        == 2
      if (A[i] > A[i+1]) the A[i] <-> A[i+1]; == 3
  }
}
```

* 수행시간
  * 1의 for 루프는 n-1번 반복
  * 2의 for 루프는 각각 n-1, n-2, … , 2, 1번 반복
  * 3의 교환은 상수시간 작업
* T(n) = (n-1) + (n-2) + … + 2 + 1 = O(n<sup>2</sup>)
  * 최악, 최선, 평균 항상 n(n-1) / 2번의 비교연산을 수행하게 되므로 O(n<sup>2</sup>)이다.

### Insertion Sort

* 맨 처음 인덱스에 있는 원소를 정렬되어있는 상태라고 보고, 두번째 인덱스에 있는 데이터를 이 정렬된 배열에 삽입하면서, 두개의 데이터가 다시 정렬된 상태가 되도록 만드는 정렬 방법이다. 반복적으로.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/sort_03.png?raw=true)







