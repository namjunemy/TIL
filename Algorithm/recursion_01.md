> 인프런 - 부경대IT융합응용공학과 궘오흠 교수님의 '**영리한 프로그래밍을 위한 알고리즘 강좌** '([링크](https://www.inflearn.com/course/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B0%95%EC%A2%8C/))와 '**쉽게 배우는 알고리즘** 관계 중심의 사고법 - 문병로' 참조

# 2-1. 순환(Recursion)의 개념과 기본 예제 1

## Recursion

우리말로 순환 또는 재귀함수 라고 한다. 즉, 자기 자신을 호출하는 함수, 메소드를 뜻한다.

```java
void func(...) {
  ...
  func(...);
  ...
}
```

### ex1 - 무한루프

```java
public class Code01 {
  public static void main(String[] args) {
    func();
  }
  
  private static void func() {
    System.out.println("Hello...");
    func();
  }
}
```

```java
Hello...
Hello...
Hello...
Hello...
Hello...
...
```

### ex2 - 항상 무한루프에 빠질까?

* recursion이 적절한 구조를 가지고 있다면, 항상 무한루프에 빠지는 것은 아니다.
  * **Base case** : 적어도 하나의 recursion에 빠지지 않는 경우가 존재해야 한다.
  * **Recursive case** : recursion을 반복하다보면 결국 Base case로 수렴해야 한다.

```java
public class Code2 {
  public static void main(String[] args) {
    int n = 4;
    func(n);
  }

  private static void func(int n) {
    if (n <= 0)
      return;
    else {
      System.out.println("Hello...");
      func(n - 1);
    }
  }
}
```

```java
Hello...
Hello...
Hello...
Hello...

Process finished with exit code 0
```

### ex3 - 1~n 까지의 합

* 입력으로 받은 정수 n에 대해서 1~ n까지의 합을 구한다.

```java
public class Code03 {
  public static void main(String[] args) {
    int result = func(4);
    System.out.println("result: " + result);
  }

  private static int func(int n) {
    if (n == 0)
      return 0;
    else
      return n + func(n - 1);
  }
}
```

```java
result: 10

Process finished with exit code 0
```

### recursion의 해석

* 이 함수의 mission은 0~n까지의 합을 구하는 것이다.
* n = 0이라면 합은 0이다.
* n이 0보다 크다면 0에서 n까지의 합은 0에서 n-1까지의 합에 n을 더한 것이다.

```java
public static int func(int n) {
  if(n == 0) 
    return 0;
  else
    return n + func(n - 1);
}
```

### 순환함수와 수학적귀납법

* 정리 
  * func(int n)은 음이 아닌 정수 n에 대해서 0에서 n까지의 합을 올바로 계산한다.
* 증명
  1. n=0인 경우 : n=0인 경우 0을 반환한다. 올바르다.
  2. 임의의 양의 정수 k에 대해서 n<k인 경우 0에서 n까지의 합을 올바르게 계산하여 반환한다고 가정하자.
  3. n=k인 경우를 고려해보자. func메소드는 먼저 func(k-1)을 호출하는데 2번에 가정에 의해서 0에서 k-1까지의 합이 올바로 계산되어 반환된다. 메소드 func는 그 값에 n을 더해서 반환한다. 따라서 메소드 func는 0에서 k까지의 합을 올바로 계산하여 반환한다. 
* 항상 이런식으로 증명을 작성할 필요는 없지만, recursion에 대해서 고민할 때 위와 같은 방식으로 고민을 하는것이 상당한 도움이 된다.

## Factorial: n!

* 0! = 1
* n! = n x (n-1)!  n>0
* Factorial이 가진 recursive한 성질을 그대로 이용하여 아래와 같은 코드를 작성할 수 있다.

```Java
public static int factorial(int n) {
  if(n == 0)
    return 1;
  else
    return n * factorial(n - 1);
}
```

### 순환함수와 수학적귀납법

* 정리
  * factorial(int n)은 음이 아닌 정수 n에 대해서 n!을 올바로 계산한다.
* 증명
  1. n=0인 경우 : n=0인 경우 1을 반환한다. 올바르다.
  2. 임의의 양의 정수 k에 대해서 n<k인 경우 n!을 올바르게 계산한다고 가정하자.
  3. n=k인 경우를 고려해보자. factorial은 먼저 factorial(k-1) 호출하는데 2번의 가정에 의해서 (k-1)!이 올바로 계산되어 반환된다. 따라서 메소드 factorial은 k *(k-1)! = k! 을 반환한다.

## X<sup>n</sup>

* X<sup>0</sup> = 1
* X<sup>n</sup> = X*X<sup>n-1</sup>   if n>0

```java
public static double power(double x, int n) {
  if (n == 0)
    return 1;
  else
    return x * power(x, n-1);
}
```

## Fibonacci Number

* f<sub>0</sub> = 0
* f<sub>1</sub> = 1
* f<sub>n</sub> =  f<sub>n-1</sub> + f<sub>n-2</sub>  n>1

```java
public int fibonacci(int n) {
  if(n < 2) 
    return n;
  else
    return fibonacci(n - 1) + fibonacci(n - 2);
}
```

## 최대 공약수: Euclid Method

* m >= n인 두 양의 정수 m과 n에 대해서 m이 n의 배수이면 gcd(m, n) = n이고,
* 그렇지 않으면 gcd(m, n) = gcd(n, m % n)이다.
  * n과 m을 n으로 나눈 나머지 간의 최대공약수이다.

```Java
public static int gcd(int m, int n) {
  if (m < n) {
    int tmp = m;
    m = n;
    n = tmp;
  }
  if (m % n == 0)
    return n;
  else
    return gcd(n, m % n);
}
```

### Euclid Method : 좀 더 단순한 버전

* gcd(p, q)
  * if q = 0 -> p
  * otherwise gcd(q, p % q)

```java
public static int gcd(int p, int q) {
  if (q == 0)
    return p;
  else
    return gcd(q, p % q);
}
```

### 최대공약수 최소공배수 예제(유클리드 호제법)

```java
public class Code04 {
  public static void main(String[] args) {
    int result = gcd(12, 6);
    System.out.println("최대공약수: " + result);
    result = euclidGcd(12, 6);
    System.out.println("유클리드 호제법 최대공약수: " + result);
    result = lcm(12, 8);
    System.out.println("최소공배수: " + result);
  }

  private static int gcd(int m, int n) {
    if (m < n) {
      int tmp = m;
      m = n;
      n = tmp;
    }
    if (m % n == 0)
      return n;
    else
      return gcd(n, m % n);
  }

  private static int euclidGcd(int p, int q) {
    if (q == 0)
      return p;
    else
      return euclidGcd(q, p % q);
  }

  private static int lcm(int m, int n) {
    int bigger = 0;
    bigger = (m > n) ? m : n;
    while (true) {
      if ((bigger % m == 0) && (bigger % n == 0))
        return bigger;
      bigger++;
    }
  }
}
```

```java
최대공약수: 6
유클리드 호제법 최대공약수: 6
최소공배수: 24

Process finished with exit code 0
```