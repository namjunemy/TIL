> 인프런 - 부경대IT융합응용공학과 궘오흠 교수님의 '**영리한 프로그래밍을 위한 알고리즘 강좌** '([링크](https://www.inflearn.com/course/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B0%95%EC%A2%8C/))와 '**쉽게 배우는 알고리즘** 관계 중심의 사고법 - 문병로' 참조

# Dynamic Programming 04

Matrix-Chain Multiplication

## 행렬의 곱셈

* p x q 행렬 A와 q x r 행렬 B 곱하기
  * 곱셈 연산의 횟수 = pqr

```java
void prodict(int A[][], int B[][], int C[][]) {
  for (int i = 0; i < p; i++) {
    for (int j = 0; j < r; j++) {
      C[i][j] = 0;
      for (int k = 0; k < q; k ++)
        C[i][j] += A[i][k] * B[k][j];
    }      
  }
}
```

## Matrix-Chain 곱하기

* 행렬 A는 10 x 100, B는 100 x 5, C는 5 x 50
* 세 행렬의 곱 ABC는 두 가지 방법으로 계산가능 (결합법칙이 성립, 교환법칙은 X)
  * (AB)C : 7,500번의 곱셈이 필요(10x100x5 + 10x5x50)
  * A(BC) : 75,000번의 곱셈이 필요(100x5x50 + 10x100x50)
* 즉 곱하는 순서에 따라서 연산량이 다름
* n개의 행렬의 곱 A1A2A3...An을 계산하는 최적의 순서는?
* 여기서 Ai는 Pk-1 x Pk 행렬이다.

### Optimal Substructure

* 동적계획법은 Optimal Substructure를 순환식으로 풀어서  그것을 Memoization이나 Bottom-Up 방식으로 푸는 방법이므로, 우리는 Optimal Substructure를 확인하는 것 먼저 시작하면 된다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_13.png?raw=true)

### 순환식

* 위의 Optima Substructure에서 X가 m[i, k], Y가 m[k + 1, j], X*Y 즉, Z가 Pi-1PkPj에 대응된다.
* 순환식의 경우 항상 basecase가 존재해야 하고, 그것은 i = j 인 경우 이다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_14.png?raw=true)

### 계산 순서

* 아래의 그림에서 bottom-up 방식으로 계산하여 m(i, j) = (3, 7)의 값을 알려면,
* 일단 k가 i ~ j-1 까지의 범위를 가지니까, m(i, i) ~ m(i, j-1) 까지의 값을 다 알아야 한다.
* 마찬가지로 세로 화살표에 해당하는 m(i+1, j) ~ m(j, j)까지의 값을 다 알아야 한다.
* 이 두가지 를 이용해서 m(3, 7)의 값을 구할 수 있다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_15.png?raw=true)

- Bottom-up 방식으로 계산하려면 아래의 그림에서 밑에서부터 올라오면서(위를 향하는 화살표의 순서로) 한칸씩 값을 구하는 방법이 한가지 있고,

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_16.png?raw=true)

- 오른쪽 아래 대각선 순서로 한칸씩 값을 구하는 방법이 한가지 있다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_17.png?raw=true)

### 동적계획법

* 오른쪽 아래 대각선 순서로 계산하는 코드이다.
* 시간복잡도는 Θ(n^3)이다.
* 01 - 02 : (i, i)대각선을 0으로 채우고 시작한다.
* 03 : (i,i) 대각선을 제외한 대각선의 갯수가 n-1개 이다.
* 04 : 각 대각선에서 계산해야 할 칸의 갯수이다. r이 1일 때 n - 1개, r이 2일 때 n - 2개.
* 05 - 10 : r번째 대각선의 i번째 값을 구하면 된다.
  * i값은 +1씩 증가하고, j값은 i+r로 증가한다.
  * 다음으로 순환식을 이용해서 m\[i\]\[j\]를 구한다. Bottom-up방식에서는 항상 이전의 값이 보장된 상태에서 다음 값을 구하기 때문에 순환식을 그대로 이용하면 된다.
  * k는 i+1부터 j-1까지 계산한 값이 현재의 m\[i\]\[j\]값 보다 작으면 그것이 새로운 m\[i\]\[j\] 값이 된다.(최소값을 구하는 로직이므로)

```java
00 int matrixChain(int n) {
01   for (int i = 1; i <= n; i++)
02     m[i][i] = 0;
03   for (int r = 1; r <= n - 1; r++) {
04     for (int i = 1; i <= n - r; i++) {
05       int j = i + r;
06       m[i][j] = m[i+1][j] + p[i-1]*p[i]*p[j];
07       for (int k = i + 1; k <= j - 1; k++) {
08         if (m[i][j] > m[i][k] + m[k+1][j] + p[i-1]*p[k]*p[i])
09           m[i][j] = m[i][k] + m[k+1][j] + p[i-1]*p[k]*p[i];
10       }
11     }
12   }
13   return m[1][n];
14 }
```





