# 2-2. 순환(Recursion)의 개념과 기본 예제 2

## Recursive Thinking - 순환적으로 사고하기

### Recursion은 수학함수 계산에만 유용한가?

* 수학함뿐 아니라 다른 많은 문제들을 recursion으로 해결할 수 있다.

### 문자열의 길이 계산

* 순서대로 앞에서 부터 하나씩 카운트 한다.
* 또는, 총 문자열의 길이는 첫번째 문자를 뺀, 전체 문자열의 길이 + 1(첫번째 문자)이다.(recursive thinking)

```java
if the string is empty	//base case
  return 0;
else
  return 1 plus the length of the string that excludes the first character;
```

```java
public static int length(String str) {
  if (str.equals(""))
    return 0;
  else
    return 1 + length(str.substring(1));
}
```

#### substring의 정의

```java
public String substring(int start)
```

Returns a new `String` that contains a subsequence of characters currently contained in this character sequence. The substring begins at the specified index and extends to the end of this sequence.

- Parameters:

  `start` - The beginning index, inclusive.

- Returns:

  The new string.

- Throws:

  `StringIndexOutOfBoundsException` - if `start` is less than zero, or greater than the length of this object.

### 문자열의 프린트

```java
public static void printChars(String str) {
  if(str.length() == 0)
    return;
  else {
    System.out.print(str.charAt(0));
    printChars(str.substring(1));
  }
}
```

### 문자열을 뒤집어 프린트

* recursive하게 프린트 하려면
  * 먼저 첫 문자를 뺀 나머지 문자열을 뒤집어 프린트 한 후
  * 마지막으로 첫 문자를 프린트 한다.

```java
public static void printCharsReverse(String str) {
  if(str.length() == 0)
    return;
  else {
    printCharsReverse(str.substring(1));
    System.out.print(str.charAt(0));
  }
}
```

### 2진수로 변환하여 출력

* 음이 아닌 정수 n을 이진수로 변환하여 인쇄한다.
* n을 2로 나눈 몫을 먼저 2진수로 변환하여 인쇄한 후
* n을 2로 나눈 나머지를 인쇄한다.

```java
public void printInBinary(int n) {
  if (n < 2)
    System.out.print(n);
  else {
    printInBinary(n / 2);
    System.out.print(n % 2)
  }
}
```

### 배열의 합 구하기

* data[0]에서 data[n-1]까지의 합을 구하여 반환한다.

```java
public static int sum(int n, int[] data) {
  int (n <= 0)
    return 0;
  else
    return sum(n-1, data) + data[n - 1];
}
```

### 데이터파일로부터 n개의 정수 읽어오기

* Scanner in이 참조하는 파일로부터 n개의 정수를 입력받아 배열 data의 data[0], ... , data[n-1]에 저장한다.

```java
public void readFrom(int n, int[] data, Scanner in) {
  if (n == 0)
    return;
  else {
    readFrom(n-1, data, in);
    data[n-1] = in.nextInt();
  }
}
```

## Recursive VS Iteration

* 모든 순환함수는 반복문(Iteration)으로 변경 가능
* 그 역도 성립함. 즉, 모든 반복문은 recursion으로 표현 가능함
* 순환함수는 복잡한 알고리즘을 단순하고 알기쉽게 표현하는 것을 가능하게 함
* 하지만 recursion은 함수 호출에 따른 오버헤드가 있음(매개변수 전달, 액티베이션 프레임 생성 등)