## 알고리즘을 위한 자바 IO

> codeplus - 프로그래밍 대회에서 사용하는 Java 참고

### System.out

* System.out.println();
* System.out.printf("%d", n)
  * 실수형, 문자형 자료 출력 가능

### Scanner

* next[자료형]을 이용해서 입력을 받을 수 있고,
* hasNext[자료형]을 이용해서 입력받을 수 있는 자료형이 있는지 구할 수 있다.
* 두 수 입력

```java
public class Main {
  public static void main(String[] args) {
    Scanner scanner = new Scanner(System.in);
    int a, b;
    a = scanner.nextInt();
    b = scanner.nextInt();
    System.out.println(a + b);
  }
}
```

* 입력에서 정수가 주어지는 동안 계속 입력 받음

```java
public class Main {
  public static void main(String[] args) {
    Scanner scanner = new Scanner(System.in);
    int sum = 0;
    while(scanner.hasNextInt()) {
      sum += scanner.nextInt();
    }
    System.out.println(sum);
  }
}
```

* 정수와 문자열 동시 처리
  * 1의 뒤에 줄바꿈 \n이 존재하기 때문에, 줄바꿈을 읽어들여서 hi를 읽지 못한다.
  * 따라서, nextLine()으로 다음 문장을 읽기 전에, scanner.nextLine(); 줄바꿈을 입력받는 코드를 한 줄 작성해야 올바른 의도대로 코딩을 할 수 있다..

```java
<입력>
1
hi
```

```java
public class Main {
  public static void main(String[] args) {
    Scanner scanner = new Scanner(System.in);
    int n = scanner.nextInt();
    scanner.nextLine();
    String s = scanner.nextLine();
    System.out.println(n + "\n" + s);
  }
}
```

### BufferedReader

* Scanner는 매우 편리하지만 속도가 느리기 때문에, 입력을 많이 받아야 하는 경우에는 BufferedReader를 사용하는 것이 훨씬 좋다.
* bufferedReader에서는 read와 readLine만 있기 때문에, 정수는 파싱을 해야한다.

```java
public class Main {
  public static void main(String[] args) throws IOException {
    BufferedReader bf = new BufferedReader(new InputStreamReader(System.in));
    String[] line = bf.readLine().split(" ");
    String a = line[0] + line[1];
    String b = line[2] + line[3];
    long result = Long.valueOf(a) + Long.valueOf(b);
    System.out.println(result);
  }
}
```

```java
10 20 30 40
4060
```

### StringTokenizer

* 문자열을 토큰으로 잘라야 할 때 사용하면 편하다.
* 수 N개의 합을 구하는 문제
  * 입력받은 수 N개의 합을 출력한다.
  * 예제입력

```java
1 2 3 4 5
```

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class Main {
  public static void main(String[] args) throws IOException {
    BufferedReader bf = new BufferedReader(new InputStreamReader(System.in));
    String line = bf.readLine();
    StringTokenizer st = new StringTokenizer(line, " ");
    int sum = 0;
    while(st.hasMoreTokens())
      sum += Integer.valueOf(st.nextToken());
    System.out.println(sum);
  }
}
```

```java
15
```

* 문자열 S에 포함되어 있는 자연수의 합을 출력하라
  * 입력

```java
10,20,30,40,50
```

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class Main {
  public static void main(String[] args) throws IOException {
    BufferedReader bf = new BufferedReader(new InputStreamReader(System.in));
    String line = bf.readLine();
    StringTokenizer st = new StringTokenizer(line, ",");
    int sum = 0;
    while(st.hasMoreTokens())
      sum += Integer.valueOf(st.nextToken());
    System.out.println(sum);
  }
}
```

```java
150
```

### StringBuilder

* 출력해야 하는 것이 많은 경우에는, 매번 출력하는 것 보다
* StringBuilder를 이용해 문자열을 만들고, 한번에 출력하는 것이 속도면에서 좋다.

```java
import java.util.Scanner;

public class Main {
  public static void main(String[] args) {
    Scanner sc = new Scanner(System.in);
    int a = sc.nextInt();
    for (int i = 1; i <= a; i++)
      System.out.println(i);
  }
}

```

```java
수행시간 : 676 MS, 메모리 : 30256 KB
```

```java
import java.util.Scanner;

public class Main {
  public static void main(String[] args) {
    Scanner sc = new Scanner(System.in);
    int a = sc.nextInt();
    StringBuilder sb = new StringBuilder();
    for (int i = 1; i <= a; i++)
      sb.append(i + "\n");
    System.out.println(sb);
  }
}
```

```java
수행시간 : 216 MS, 메모리 : 11532 KB
```

