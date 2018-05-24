> 인프런 - 부경대IT융합응용공학과 궘오흠 교수님의 '**영리한 프로그래밍을 위한 알고리즘 강좌** '([링크](https://www.inflearn.com/course/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B0%95%EC%A2%8C/))와 '**쉽게 배우는 알고리즘** 관계 중심의 사고법 - 문병로' 참조

# Dynamic Programming 01

동적계획법

## Motivation - Fibonacci Numbers

* 피보나치 수열의 합을 계산하는 간명한 방법은 recursion을 이용하는 것이다.
* 하지만, recursion을 이용하면 많은 중복이 발생하게 된다. 올바른 방법이긴 하지만 매우 비효율적인 것이다.
  * fib(7)에서 fib(6)과 fib(5)를 계산했지만, fib(6)이 재귀 호출될 때 fib(5)를 다시 구하게 된다.

```java
int fib(int n) {
  if (n == 1 || n == 2)
    return 1;
  else
    return fib(n - 2) + fib(n - 1);
}
```

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_01.png?raw=true)

### Memoization

* 메모이제이션이라고 하는 이 방법은 중복되는 계산의 결과를 기억해놓자는 의미이다.
* 아주 간단한 방법이다. 중간 계산 결과를 배열에 저장함으로써 중복계산을 피한다. 이를 caching이라고 한다.
* 따라서, 중간 계산 결과가 저장되어 있다면, f[n]을 다시 계산할 필요가 없이 리턴한다.
* f[n]이 저장되어 있지 않다면, recursion으로 계산하게 된다. 하지만, 어떤 경우에도 중복 계산은 없어진다.

```java
int fib(int n) {
  if (n == 1 || n == 2)
    return 1;
  else if (f[n] > -1)  // 배열 f가 -1로 초기화되어 있다고 가정
    return f[n];       // 즉, 이미 계산되어 저장된 값이라는 의미
  else {
    f[n] = fib(n - 2) + fib(n - 1);  //중간 계산 결과를 caching
    return f[n];
  }
}
```

| 배열 f |  1   |  2   |  3   |  4   |  5   |  6   |  7   |  8   |  9   |
| :----: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|   값   |  1   |  1   |  2   |  3   |  5   |  8   |  13  |  21  |  34  |

### Dynamic Programming(Bottom-up 방식)

* Bottom-up 방식으로 순차적으로 계산해 나가면서 중복 계산을 피할 수 있다.

```java
public static int fib(int n) {
  int[] f = new int[n + 1];
  f[1] = f[2] = 1;
  for (int i = 3; i <= n; i++)
    f[i] = f[i - 1] + f[i - 2];
  return f[n];
}
```

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_02.png?raw=true)

## Memoization VS Dynamic Programming

* 순환식의 값을 계산하는 기법들이다.
* 둘 다 동적계획법의 일종으로 보기도 한다.
* Memoization은 top-down 방식이며, 실제로 필요한 sub problem만을 푼다.
  * 중복계산은 없지만 resursion이고, 계속해서 함수를 호출하기 때문에 for문을 반복하는 시간보다는 함수의 수행시간이라는 측면에서 overhead가 수반된다.
* 동적계획법은 bottot-up 방식이며, resursion에 수반되는 overhead가 없다.