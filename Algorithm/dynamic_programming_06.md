> 인프런 - 부경대IT융합응용공학과 궘오흠 교수님의 '**영리한 프로그래밍을 위한 알고리즘 강좌** '([링크](https://www.inflearn.com/course/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B0%95%EC%A2%8C/))와 '**쉽게 배우는 알고리즘** 관계 중심의 사고법 - 문병로' 참조

# Dynamic Programming 06

Knapsack Problem

## Knapsack

* n개의 아이템과 배낭
* 각각의 아이템은 무게 Wi와 가격 Vi를 가짐
* 배낭의 용량 W
* 목적: **배낭의 용량을 초과하지 않으면서 가격이 최대가 되는 부분집합**
* 예:
  * {1, 2, 5}는 가격의 합이 35
  * {3, 4}는 가격의 합이 40
  * {3, 5}는 46이지만 배낭의 용량을 초과함
* item table
  * knapsack instance(weight limit W = 11)

| i    | Vi   | Wi   |
| ---- | ---- | ---- |
| 1    | 1    | 1    |
| 2    | 6    | 2    |
| 3    | 18   | 5    |
| 4    | 22   | 6    |
| 5    | 28   | 7    |

### Greedy

* 가격이 높은 것 부터 선택
* 무게가 가벼운 것부터 선택
* 단위 무게당 가격이 높은것 부터 선택
* 위의 단순한 접근으로는 이 문제를 해결할 수 없다.

### 순환식

* OPT(i) : 아이템 1, 2, … , i로 얻을 수 있는 최대 이득
  * 경우 1 : 아이템 i를 선택하지 않는 경우
    * OPT(i) = OPT(i - 1)
  * 경우 2 : 아이템 i를 선택하는 경우
    * OPT(i) = ?
* OPT(i, w) : 배낭 용량이 w일 때 아이템 1, 2, … , i로 얻을 수 있는 최대 이득
  * 경우 1 : 아이템 i를 선택하지 않는 경우
    * i번째 아이템의 Wi가 현재 배낭의 크기 W보다 크다면 선택할 수 없으므로 해당 아이템은 지나친다.
    * OPT(i, w) = OPT(i - 1, w)
  * 경우 2 : 아이템 i를 선택하는 경우
    * 항상 이 경우를 선택할 수 있는 것이 아니라 현재 배낭의 크기 W가 Wi보다 크거나 같을 때만 이 경우를 선택할 수 있다.
    * OPT(i) = OPT(i - 1, W - Wi) + Vi

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_23.png?raw=true)

### 풀이

* 순환식에서 보면 bottom-up방식으로 표현하기 위한 순서도 간단하다. i를 구하기 위해서 i-1을 알면 되므로 행우선 순서로 하나씩 구하면 된다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_24.png?raw=true)

* 따라서 코드의 구조도 간단하다.
  * pesudo code

```java
KNAPSACK(n, W, (w1, w2, ... , wn), (v1, v2, ... , vn))
  
FOR w = 0 TO W
  M[0, w] <- 0

FOR i = 1 TO n
  FOR w = 1 TO W
   IF (wi > w) M[i, w] <- M[i-1, w],
   ELSE        M[i, w] <- max {M[i-1, w], vi + M[i-1, w-wi]}

RETURN M[n, W]
```

### 시간복잡도

* 시간복잡도 O(nW)
* 다항시간인가?