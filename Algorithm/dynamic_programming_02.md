> 인프런 - 부경대IT융합응용공학과 궘오흠 교수님의 '**영리한 프로그래밍을 위한 알고리즘 강좌** '([링크](https://www.inflearn.com/course/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B0%95%EC%A2%8C/))와 '**쉽게 배우는 알고리즘** 관계 중심의 사고법 - 문병로' 참조

# Dynamic Programming 02

동적계획법 basic example 02

## 행렬 경로 문제

* 정수들이 저장된 n x n 행렬의 좌상단에서 우하단까지 이동한다. 단 오른쪽이나 아래쪽 방향으로만 이동할 수 있다.
* 방문한 칸에 있는 정수들의 합이 최소화되도록 하라.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_03.png?raw=true)

### Key Observation

* (i, j)에 도달하기 위해서는 (i, j-1) 혹은 (i-1, j)를 거쳐야 한다.(오른쪽이나 아래방향으로만 이동하므로)
* 또한, (i, j-1) 혹은 (i-1, j)까지는 최선의 방법으로 이동해야 한다.

### 순환식

* i와 j가 1인 경우는 거쳐온 칸이 하나밖에 있을 수 없음을 주의

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_04.png?raw=true)

### Recursive Algorithm

* 위의 순환식을 토대로 알고리즘 작성

```java
int mat(int i, int j) {
  if (i == 1 && j == 1)
    return m[i][j];
  else if (i == 1)
    return mat(1, j-1) + m[i][j];
  else if (j == 1)
    return mat(i-1, 1) + m[i][j];
  else
    return Math.min(mat(i-1, j), mat(i, j-1)) + m[i][j];
}
```

* 위의 순환식을 그대로 recursion으로 옮겨서 계산하면 매우 많은 계산의 중복이 일어난다. 따라서, Memoization이나 Bottom-up방식을 이용해서 중복을 줄일 수 있다.

#### Memoization

* 2차원 배열을 새로 만들고, 거기에 계산된 값을 저장한다. 모든 칸은 -1로 초기화 했다고 가정한다.

```java
int mat(int i, int j) {
  if (L[i][j] != -1) return L[i][j];
  if (i == 1 && j == 1)
    L[i][j] = m[i][j];
  else if (i == 1)
    L[i][j] = mat(1, j-1) + m[i][j];
  else if (j == 1)
    L[i][j] = mat(i-1, 1) + m[i][j];
  else
    L[i][j] Math.min(mat(i-1, j), mat(i, j-1)) + m[i][j];
  return L[i][j];
}
```

#### Bottom-Up

* 오른쪽이나 아래방향으로만 이동할 수 있다는 조건을 위반하지 않는 순서로 계산이 가능하다.
* 간단하게 행우선(행 1->4, 열 1->4)순으로 계산해볼 수 있다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_05.png?raw=true)

* 행우선으로 항상 전칸이 계산되기 때문에 recursion으로 호출하지 않아도 되고, 시간복잡도는 O(n^2)이 된다.

```java
int mat(int i, int j) {
  for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= n; j++) {
      if (i == 1 && j == 1)
        L[i][j] = m[1][1];
      else if (i == 1)
        L[i][j] = mat(1, j-1) + m[i][j];
      else if (j == 1)
        L[i][j] = mat(i-1, 1) + m[i][j];
      else
        L[i][j] Math.min(mat(i-1, j), mat(i, j-1)) + m[i][j];
    }
  }
  return L[n][n];
}
```

#### Common Trick

* 위의 코드에서 분기문이 4개로 나뉘는 것은 i나j가 1인경우 i-1, j-1이 0이 되서 유효하지 않은 값이 되기 때문이다.
* 보통 이런 경우 코드를 간결하게 만드는 trick을 쓸 수 있다.
* 2차원 배열을 만들때, 거의 모든 프로그래밍언어에서 0부터 시작을 하므로 실제로는 1 ~ n 전에 0행 0열이 존재하게 된다.
* 이 0행 0열에 모든 값을 ∞로 채워 넣으면 둘중에 작은 값을 선택할 때, 무시할 수 있게 된다.(작은 것을 고르는데 무한대를 고르는 경우는 없으므로)
* 따라서, 분기문을 합칠 수 있다.

```java
int mat(int i, int j) {
  for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= n; j++) {
      if (i == 1 && j == 1)
        L[i][j] = m[1][1];
      else
        L[i][j] Math.min(mat(i-1, j), mat(i, j-1)) + m[i][j];
    }
  }
  return L[n][n];
}
```

### 경로 구하기

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_06.png?raw=true)

* 위와 같이 경로를 구하려면, 별도의 작업 없이 Bottom-Up 방식으로 계산할 때, 배열 P의 값을 저장해 주면 된다.

```java
int mat(int i, int j) {
  for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= n; j++) {
      if (i == 1 && j == 1) {
        L[i][j] = m[1][1];
        P[i][j] = '-';
      }
      else {
        if (L[i-1][j] < L [i][j-1]) {
          L[i][j] = m[i][j] + L[i-1][j];
          P[i][j] = '↑';
        } else {
          L[i][j] = m[i][j] + L[i][j-1];
          P[i][j] = '←';
        }
      }
    }
  }
  return L[n][n];
}
```

* 경로 출력하는 함수

```java
void pringPath() {
  int i = n, j = n;
    while (P[i][j] != '-') {
      print(i + " " + j);
      if (P[i][j] == '←')
        j = j-1;
      else
        i = i-1;
    }
    print(i + " " + j);
}
```

* recursive하게 경로 출력하기
  * (i, j)에 해당하는 경로를 출력하려면 (i, j-1) 또는 (i-1, j)까지의 경로를 출력한 다름 (i, j)를 출력하면 된다.

```java
void pringPathRecursive(int i, int j) {
  if (P[i][j] == '-')
    print(i + " " + j);
  else {
    if (P[i][j] == '←')
      printPathRecursive(i, j-1)
    else
      printPathRecursive(i-1, j)
    print(i + " " + j);
  }
}
```

