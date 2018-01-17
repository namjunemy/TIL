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
* 구현

```java
public class Selection {
  private static int[] input = {5, 6, 2, 8, 7, 23, 4, 1};

  public static void main(String[] args) {
    selectionSortMin(input, input.length);
    for (int a : input) {
      System.out.print(a + " ");
    }
  }

  private static void selectionSortMin(int[] input, int length) {
    int min;
    int tmp;
    for (int i = 0; i < length - 1; i++) {
      min = i;
      for (int j = i + 1; j < length; j++) {
        if (input[j] < input[min])
          min = j;
      }
      tmp = input[i];
      input[i] = input[min];
      input[min] = tmp;
    }
  }

  private static void selectionSortMax(int[] input, int length) {
    int max;
    int tmp;
    for (int i = length - 1; i > 0; i--) {
      max = i;
      for (int j = i -1; j >= 0; j--) {
        if (input[j] > input[max])
          max = j;
      }
      tmp = input[i];
      input[i] = input[max];
      input[max] = tmp;
    }
  }
}
```

```java
1 2 4 5 6 7 8 23 
Process finished with exit code 0
```

  

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
* 구현

```java
public class Bubble {
  private static int[] input = {5, 6, 2, 8, 7, 23, 4, 1};

  public static void main(String[] args) {
    bubbleSort(input, input.length);
    for (int a : input) {
      System.out.print(a + " ");
    }
  }

  private static void bubbleSort(int[] input, int length) {
    int tmp;
    for (int i = length - 1; i > 0; i--)
      for (int j = 0; j < i; j++) {
        if (input[j] > input[j + 1]) {
          tmp = input[j];
          input[j] = input[j + 1];
          input[j + 1] = tmp;
        }
      }
  }
}
```

```java
1 2 4 5 6 7 8 23 
Process finished with exit code 0
```

  

### Insertion Sort

* 맨 처음 인덱스에 있는 원소를 정렬되어있는 상태라고 보고, 두번째 인덱스에 있는 데이터를 이 정렬된 배열에 삽입하면서, 두개의 데이터가 다시 정렬된 상태가 되도록 만드는 정렬 방법이다. 반복적으로.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/sort_03.png?raw=true)

* insertion의 일반적인 one step
  * k-1까지 정렬된 배열에 k번째 인덱스를 추가하는데, k번째 인덱스가 위치해야할 적절한 위치에 끼워 넣어서,
  * 배열의 k번째 까지 정렬되도록 하는 흐름을 가진다. 
* 어떻게 k번째 인덱스가 들어갈 위치를 찾는가?
  * 앞에서 부터 비교하면서 찾는 방법, 뒤에서 부터 비교하면서 찾는 방법이 있다.
  * 얼핏 생각하면 같은 방법인것 처럼 보이나, 다른점이 있다.
  * 앞에서 부터 인덱스의 요소들을 비교하면서 자리를 찾는다면, k번째 인덱스보다 작은 값들을 일일히 검사를 해야한다. 이것은 뒤에서 부터 자리를 찾는 작업에서도 똑같이 이루어진다.
  * 하지만, 들어갈 위치를 찾았을 때, 배열이라는 자료구조의 특성상 해당 위치부터 k-1인덱스까지의 요소들을 한칸씩 shift 시키는 작업이 필요하다.
  * 결과적으로 앞에서부터 비교를 하면, 모든 데이터를 한번씩은 적어도 봐야된다.
  * 그러나, 뒤에서 부터 비교한다면 비교와 동시에 switch하는 작업을 행해줌으로써 자동으로 shift 작업을 수행한다.
  * 해당 데이터가 자리를 찾는 순간 앞의 데이터들은 비교할 필요가 없다. 따라서, 뒤에서 부터 비교해서 자리를 찾는 작업이 더 효율적인 작업이다.
* 비교와 동시에 switch 작업을 하기위해서 임시변수 tmp를 유지한다. 그리고 k-1 부터 0번 인덱스까지 compare작업과 shift작업을 반복적으로 수행한다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/sort_04.png?raw=true)

* pseudocode

```java
insertionSort(A[], n) { // 배열 A[1...n]을 정렬한다.
  for i <- 2 to n                       == 1
    A[1...i]의 적당한 자리에 A[i]를 삽입한다.  == 2
}
```

* 수행시간
  * 1의 for 루프는 n-1번 반복
  * 2의 삽입은 최악의 경우 i-1번 비교
* 최악의 경우
  * T(n) = (n-1) + (n-2) + … + 2 + 1 = O(n<sup>2</sup>)
* 최선의 경우 = 즉,  배열이 정렬되어있는 경우에는 O(n)의 시간 복잡도를 가진다.
* 따라서, selection sort나 bubble sort와 비교했을때, 최악의 경우에는 차이가 없지만 평균적으로는 대략 절반 정도의 정렬 시간을 필요로 한다.
* 구현

```java
public class Insertion {
  private static int[] input = {5, 6, 2, 8, 7, 23, 4, 1, 44};

  public static void main(String[] args) {
    insertionSort(input, input.length);
    for (int a : input) {
      System.out.print(a + " ");
    }
  }

  private static void insertionSort(int[] input, int length) {
    int tmp;
    for (int i = 1; i < length; i++) {
      for (int j = i; j > 0; j--) {
        if (input[j - 1] > input[j]) {
          tmp = input[j - 1];
          input[j - 1] = input[j];
          input[j] = tmp;
        }
      }
    }
  }
}
```

```java
1 2 4 5 6 7 8 23 44 
Process finished with exit code 0
```

