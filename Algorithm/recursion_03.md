# 순환(Recursion)의 개념과 기본 예제 3

## Designing Recursion - 순환 알고리즘의 설계

### 순환적 알고리즘 설계

* 적어도 하나의 base case, 즉 순환되지 않고 종료되는 case가 있어야 함

* 모든 case는 결국 base case로 수렴해야 함

* ex - 가장 단순한 경우

  ```java
  if ( ... ) {
    //basecase
  } else {
    //recursion
  }
  ```

* **암시적(implicit) 매개변수를 명시적(explicit) 매개변수로 바꾸어라!!**

### 순차탐색

이 함수의 미션은 data[0]에서 data[n-1] 사이에서  target을 검색하는 것이다. 하지만 검색 구간의 시작 인덱스 0은 보통 생략한다. **즉 0은 암시적 매개변수이다.**

```java
int search(int[] data, int n, int targert) {
  for (int i = 0; i < n; i++)
    if (data[i] == targer)
      return i;
  return -1;
}
```

### 매개변수의 명시화: 순차 탐색

* 이 함수의 미션은 data[begin]에서 data[end] 사이에서 target을 검색한다. **즉, 검색구간의 시작점을 명시적(explicit)으로 지정한다.**
* 이 함수를 search(data, 0, n-1, target)으로 호출한다면 위에 있는 순차탐색 함수와 완전히 동일한 일을 한다.

```java
int search(int[] data, int begin, int end, int targer) {
  if (begin > end)
    return -1;
  else if (target == data[begin])
    return begin;
  else
    return search(data, begin+1, end, target)
}
```

### 순차 탐색: 다른 버전

* 이 함수의 미션은 data[begin]에서 data[end] 사이에서 target을 검색한다. 즉, 검색구간의 시작점을 명시적(explicit)으로 지정한다.

```java
int search(int[] data, int begin, int end, int targer) {
  if (begin > end)
    return -1;
  else if (target == data[begin])
    return begin;
  else
    return search(data, begin, end-1, target)
}
```

### 순차 탐색: 다른 버전

- Binary search와는 다름

```java
int search(int[] data, int begin, int end, int target) {
  if (begin > end)
    return -1;
  else {
    int middle = (begin + end) / 2;
    if (data[middle] == target)
      return middle;
    int index = search(data, begin, middle-1, target);
    if (index != -1)
      return index;
    else
      return search(data, middle+1, end, target);
  }
}
```



