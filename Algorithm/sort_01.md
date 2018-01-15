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



