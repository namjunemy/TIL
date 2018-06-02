> 인프런 - 부경대IT융합응용공학과 궘오흠 교수님의 '**영리한 프로그래밍을 위한 알고리즘 강좌** '([링크](https://www.inflearn.com/course/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B0%95%EC%A2%8C/))와 '**쉽게 배우는 알고리즘** 관계 중심의 사고법 - 문병로' 참조

# Dynamic Programming 05

Longest Common Subsequence

## LCS(Longest Common Subsequence)

* \<bcdb\>는 \<a**bc**b**d**a**b**\>의 subsequence이다.
* \<bca\>는 문자열 \<a**bc**bd**a**b\>와 \<**b**d**ca**ba\>의 common subsequence이다.(공통 부분 문자열)
* Longest common subsequence(LCS)
  * common subsequence들 중 가장 긴 것이다.
  * \<bcba\> 는 \<a**bcb**d**a**b\>와 \<**b**d**c**a**ba**\>의 LCS이다.

### Optimal Substructure

* 동적계획법으로 이 문제를 풀기 위해서는 순환식을 세워야 하는 것이고,
* 순환식이라는 것은 이 문제의 optimal substructure을 구하는 것과 같다.
* 먼저, 이 문제의 어떤 optimal substructure가 존재하는가 즉, 원래 문제에 대한 부분 문제들에 대한 해를 조합해서 원래 문제의 해를 어떻게 구할것인가가 중요하다.
* 그리고 그것의 시작은 이 문제에 대한 최적해의 한 부분이 해당 부분에 대한 최적의 해인가에 대한 질문을 던져보는 것이다.
* 예를 들면, 아래의 그림에서 x, y가 입력 문자열이고 z가 LCS일 때, LCS의 일부분이 x, y의 어떤 부분에 대한 최적해인가에 대한 질문을 던져본다.
* LCS의 마지막 문자가 A라면 x와 y의 어디엔가도 A가 존재할 것이다.
* 그렇다면, LCS에서 A를 제외한 나머지 보라색 부분의 정체는?
  * x에서 A의 앞부분인 x'와 y에서 A의 앞부분인 y'의 LCS라고 할 수 있다. 이 순환적인 특성이 성립한다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_18.png?raw=true)

### 순환식

* L[i, j] : 문자열 X = \<x1x2x3 … xi\>와 Y = \<y1y2y3 … yj\>의 LCS의 길이
* 경우 1 : Xi = Yj
  * L[i, j] = L[i - 1, j - 1] + 1

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_19.png?raw=true)

* 경우 2 : Xi != Yj
  * L[i, j] = max(L[i - 1, j], L[i, j - 1])

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_20.png?raw=true)

* 순환식
  * L[i, j] : 문자열 X = \<x1x2x3 … xi\>와 Y = \<y1y2y3 … yj\>의 LCS의 길이
  * general case에서 recursion을 반복하면 basecase에 도달하게 되므로, 완전한 순환식이다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_21.png?raw=true)

### 풀이

* 순환식을 세웠으므로 memoization이나 bottom-up방식으로 풀면 된다.
* 2차원 table을 만들어서 L[i, j]를 표현하고, 최종적으로 원하는 목표는 주어진 두 문자열의 길이가 m, n이라고 했을 때 L(m, n)을 구하는 것이다.
* 그리고 위의 순환식을 보면 L(i, j)를 구하기 위해서는 L(i, j), L(i-1, j), L(i, j-1)을 알아야 한다.
* 이 조건을 만족하는 bottom-up 계산 순서는 가장 평범하게 행우선 순서로  한 행씩 순차적으로 계산하면 된다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_22.png?raw=true)

### 동적계획법

* 0행 0열은 0으로 초기화 하고 시작한다.
* 평범한 행 우선 순서로 계산하며, 순환식을 그대로 구현한다.
* 최종 목표는 c\[m\]\[n\]을 구현하는 것이다.
* 시간복잡도는 Θ(mn)이다.

```java
int lcs(int m, int n) {
  for (int i = 0; i <= m; i++)
    c[i][0] = 0;
  for (int j = 0; j <= n; j++)
    c[0][j] = 0;
  for (int i = 0; i <= m; i++) {
    for (int j = 0; j <= n; j++) {
      if (x[i] == y[j])
        c[i][j] = c[i - 1][j - 1] + 1;
      else
        c[i][j] = Math.max(c[i - 1][j], c[i][j - 1]);
    }    
  }
  return c[m][n];
}
```

