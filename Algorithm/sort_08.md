# 3-8. Radix Sort

* Counting sort와 마찬가지로 non-comparison Sort이다.
* Radix Sort의 기본가정
  * ex) n개의 d자리 정수들 - 길이가 d인 문자열이며 각각의 문자들이 가질수 있는 값의 갯수가 상수개 이다.
  * 꼭, 정수일 필요는 없으며 길이가 d인 영문자도 가능하다.
* 가장 낮은 자리수부터 정렬한다.
  * d=3, n=7
  * 각각의 단계가 진행될 때, 반드시 **stable**해야 한다.
    * "입력에 동일한 값이 있을때 입력에 먼저 나오는 값이 출력에서도 먼저 나온다."라는 성질을 만족해야 한다.
  * 예를 들어서, 두번째 단계에서 세번째 단계로 넘어갈 때, 720 다음에 329가 와야한다. 순서가 바뀌면 stable하지 않게 되고, radix sort는 성립하지 않게 된다.
  * 매 단계에서 사용하는 정렬 알고리즘은 Stable Sort여야 한다!
  * 각 자리마다 정렬하는데 counting sort를 사용한다. (0 ~ 9까지)
  * 십의자리를 정렬하는데, 십의자리와 일의자리까지 정렬되는 것은, 매 단계가 Stable하게 정렬되기 때문이다. 이것이 Radix Sort의 핵심이다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/radix_sort_01.png?raw=true)

### pseudo code

* A배열은 d자리 정수들의 배열이다.
  * stable sort 알고리즘을 Counting Sort를 쓴다면, 이것의 시간복잡도는 n+k였으므로,(n은 데이터의 갯수, k는 데이터들이 가질 수 있는 값의 수. 여기서 k는 0 ~ 9이므로 10, 알파벳이라면 26)
  * 시간복잡도는 O(d(n+k))이며, k << n이면 O(dn)이고, d가 n보다 작고 상수라면 시간복잡도는 O(n)으로 볼 수 있다.
  * 따라서, k << n 이며 d가 상수라는 조건을 만족한다면, Radix Sort는 선형시간(linear time) 알고리즘이다.

```java
Radix-Sort(A, d)
  for i <- 1 to d
    do use a stable sort to sort array A on digit i
```

  

### 정렬 알고리즘들

* bubble, insertion, selection의 경우 최악의 경우나 평균 경우나 모두 n^2의 시간복잡도를 가진다.
* quick sort는 최악의 경우 n^2이지만 평균적으로 nlogn의 시간복잡도를 가지는 특이한 정렬 알고리즘이다.
* merge, heap sort는 둘다 최악의 경우에도 nlogn의 시간복잡도를 가지며, 이것은 최선이다. comparison sort의 시간복잡도는 nlogn보다 더 낮을 수는 없다.
* counting과 radix sort의 시간복잡도는 k와 d값에 따라 달라지는데, d가 상수이고 k << n 이라면 이것은 Θ(n)이라고 말할 수 있다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/radix_sort_02.png?raw=true)

