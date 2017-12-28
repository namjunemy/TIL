# 16장. 스트림과 병렬처리

> [소스코드 repo](https://github.com/namjunemy/this_is_java)
>
> 1절. 스트림 소개
>
> 2절. 스트림의 종류
>
> 3절. 스트림 파이프라인
>
> 4절. 필터링(distinct(), filter())
>
> 5절. 매핑(flatMapXXX(), mapXXX(), asXXXStream(), boxed())
>
> 6절. 정렬(sorted())
>
> 7절. 루핑(peek(), forEach())
>
> 8절. 매칭(allMatch(), anyMatch(), noneMatch())
>
> 9절. 기본 집계(sum(), count(), average(), max(), min())
>
> 10절. 커스텀 집계(reduce())
>
> 11절. 수집(collect())
>
> 12절. 병렬 처리

  

## 1. 스트림 소개

### 스트림은 반복자

컬렉션(배열 포함)의 요소를 하나씩 참조해서 람다식으로 처리할 수 있는 반복자이다.

* 자바 7 이전 코드

```java
List<String> list = Arrays.asList("홍길동", "신용권", "김남준");
Iterator<String> iterator = list.iterator();
while(iterator.hasNext()) {
  String name = iterator.next();
  System.out.println(name);
}
```

* 자바 8 이후 코드

```java
List<String> list = Arrays.asList("홍길동", "신용권", "김남준");
Stream<String> stream = list.stream();
stream.forEach(name -> System.out.println(name));
```

  

### 스트림 특징

* 람다식으로 요소 처리 코드를 제공한다.

  * 스트림이 제공하는 대부분의 요소 처리 메소드는 함수적 인터페이스 매개타입을 가진다.
  * 매개값으로 람다식 또는 메소드 참조를 대입할 수 있다.

  ```java
  public static void main(String[] args) {
    List<Student> list = Array.asList(
      new Student("홍길동",92),
      new Student("김남준",90)
    );
    
    Stream<Student> stream = list.stream();
    stream.forEach(s -> {
      String name = s.getName();
      int score = s.getScore();
      System.out.println(name + "-" + score);
    });
  }
  ```

  ```java
  홍길동-92
  김남준-90
  ```

* 내부 반복자를 사용하므로 병렬 처리가 쉽다.

  * 외부 반복자(external iterator)
    * 개발자가 코드로 직접 컬렉션 요소를 반복해서 요청하고 가져오는 코드 패턴
  * 내부 반복자(internal iterator)
    * 컬렉션 내부에서 요소들을 반복시키고 개발자는 요소당 처리해야할 코드만 제공하는 코드 패턴
  * 내부 반복자의 이점
    * 개발자는 요소 처리 코드에만 집중
    * 멀티코어 CPU를 최대한 활용하기 위해 요소들을 분배시켜 병렬 처리 작업을 할 수 있다.
  * 병렬(parallel) 처리
    * 한가지 작업을 서브 작업으로 나누고, 서브 작업들을 분리된 스레드에서 병렬적으로 처리한 후, 서브 작업들의 결과들을 최종 결합하는 방법
    * **자바는 ForkJoinPool 프레임워크를 이용해서 병렬 처리를 한다.**
  * 병렬 처리 코드 예

  ```java
  package sec01.stream_introduction;

  import java.util.Arrays;
  import java.util.List;
  import java.util.stream.Stream;

  public class ParallelEx {
    public static void main(String[] args) {
      List<String> list = Arrays.asList("홍길동","신용권","김남준","람다식","병렬처리");

      Stream<String> stream = list.stream();
      stream.forEach(ParallelEx::print);

      Stream<String> parallelStream = list.parallelStream();
      parallelStream.forEach(ParallelEx::print);
    }

    public static void print(String string) {
      System.out.println(string + " : " + Thread.currentThread().getName());
    }
  }
  ```

  ```java
  홍길동 : main
  신용권 : main
  김남준 : main
  람다식 : main
  병렬처리 : main
  김남준 : main
  병렬처리 : ForkJoinPool.commonPool-worker-2
  신용권 : ForkJoinPool.commonPool-worker-1
  홍길동 : ForkJoinPool.commonPool-worker-1
  람다식 : ForkJoinPool.commonPool-worker-2

  Process finished with exit code 0
  ```

* 스트림은 중간 처리와 최종 처리를 할 수 있다.

  * 중간 처리: 요소들의 매핑, 필터링, 정렬
  * 최종 처리: 반복, 카운트, 평균, 총합

  ```java
  package sec01.stream_introduction;

  import java.util.Arrays;
  import java.util.List;

  public class MapAndReduceEx {
    public static void main(String[] args) {
      List<Student> list = Arrays.asList(
          new Student("홍길동", 92),
          new Student("김남준", 90),
          new Student("김자바", 82)
      );

      double avg = list.stream()
          .mapToInt(Student::getScore)
          .average()
          .getAsDouble();

      System.out.println("평균점수: " + avg);

    }
  }
  ```

  ```java
  평균점수: 88.0

  Process finished with exit code 0
  ```

    

## 2. 스트림의 종류

### 스트림이 포함된 패키지

* 자바 8 부터 java.util.stream 패키지에서 인터페이스 타입으로 제공
  * 모든 스트림에서 사용할 수 있는 공통 메소드들이 정의 되어있는 BaseStream아래에 객체와 타입 요소를 처리하는 스트림이 있다. BaseStream은 공통 메소드들이 정의되어 있고, 코드에서 직접적으로 사용하지는 않는다.
  * BaseStream
    * Stream
    * IntStream
    * LongStream
    * DoubleStream
* 스트림 구현 객체를 얻는 방법

| 리턴타입                                     | 메소드(매개변수)                                | 소스                                       |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| Stream\<T\>                              | java.util.Collection.stream(), java.util.Collection.parallelStream() | 컬렉션                                      |
| Stream\<T\>, IntStream, LongStream, DoubleStream | Arrays.stream(T[]), Arrays.stream(int[]), Arrays.stream(long[]), Arrays.stream(double[]), Stream.of(T[]), IntStream.of(int[]), LongStream.of(long[]), DoubleStream.of(double[]) | 배열                                       |
| IntStream                                | IntStream.range(int, int), IntStream.rangeClosed(int, int) | Int 범위. rangeClosed() 메소드는 두번째 파라미터인 끝값을 포함한다. range()는 포함하지 않는다. |
| LongStream                               | LongStream.range(long, long), LongStream.rangeClosed(long, long) | long 범위. rangeClosed() 메소드는 두번째 파라미터인 끝값을 포함한다. range()는 포함하지 않는다. |
| Stream\<Path\>                           | Files.find(Path, int, BiPredicate, FileVisitOption), Files.list(Path) | 디렉토리 - Path 경로에서 BiPredicate가 true인 Path Stream을 얻을 수 있다. |
| Stream\<String\>                         | Files.lines(Path, Charset), BufferedReader.lines() | 파일 - 한 라인에 대한 텍스트 정보를 요소가지는 Stream을 얻을 수 있다. |
| DoubleStream, IntStream, LongStream      | Random.doubles(…), Random.ints(), Random.longs() | 랜덤수                                      |

  

### 컬렉션으로부터 스트림 얻기

```java
List<Student> studentList = Arrays.asList(
  new Student("홍길동", 10),
  new Student("김길동", 20),
  new Student("남길동", 30)
);

Stream<Student> stream = studentList.stream();
stream.forEach(s -> System.out.println(s.getName()));
```

```java
홍길동
김길동
남길동
```

  

### 배열로부터 스트림 얻기

```java
String[] stringArray = {"홍길동", "신용권", "김미나"};
Stream<String> stringStream = Arrays.stream(stringArray);
stringStream.forEach(a -> System.out.print(a + ","));
System.out.println();

int[] intArray = {1, 2, 3, 4, 5};
IntStream intStream = Arrays.stream(intArray);
intStream.forEach(a -> System.out.print(a + ","));
```

```java
홍길동,신용권,김미나,
1,2,3,4,5,
```

  

### 숫자 범위로부터 스트림 얻기

```java
public class FromIntRangeEx {
  public static int sum;

  public static void main(String[] args) {
    IntStream stream = IntStream.rangeClosed(1, 100);
    //IntStream stream = IntStream.range(1, 100);	//범위에 100이 포함되지 않음
    stream.forEach(a -> sum += a);
    System.out.println("총합: " + sum);
  }
}
```

```java
총합: 5050
또는
총합: 4950
```

  

### 파일로부터 스트림 얻기

```java
Path path = Paths.get("/Users/njkim/Workspace/intellij/this_is_java/ch16_스트림과병렬처리/src/sec02/kind_of_stream/linedata.txt");
Stream<String> stream;


stream = Files.lines(path, Charset.defaultCharset());
stream.forEach(System.out::println);
stream.close();
System.out.println();

File file = path.toFile();
FileReader fileReader = new FileReader(file);
BufferedReader bufferedReader = new BufferedReader(fileReader);
stream = bufferedReader.lines();
stream.forEach(System.out::println);
stream.close();
```

```java
Java8에서 추가된 새로운 기능
1. 람다식
2. 메소드 참조
3. 디폴트 메소드와 정적 메소드
4. 새로운 API 패키지

Java8에서 추가된 새로운 기능
1. 람다식
2. 메소드 참조
3. 디폴트 메소드와 정적 메소드
4. 새로운 API 패키지
```

  

### 디렉토리로부터 스트림 얻기

```java
Path path = Paths.get("/Users/njkim/Workspace/intellij/this_is_java");
Stream<Path> stream = Files.list(path);
stream.forEach(p -> System.out.println(p.getFileName()));
```

```java
ch14_람다식
ch08_인터페이스
ch15_컬렉션프레임워크
.DS_Store
out
ch07_상속
ch02_변수와타입
ch04_조건문과반복문
ch06_클래스
ch12_멀티스레드
ch03_연산자
README.md
ch05_참조타입
.gitignore
ch10_예외처리
ch16_스트림과병렬처리
ch09_중첩클래스와_중첩인터페이스
ch13_제네릭
ch11_기본_API_클래스
.git
.idea
```

  

## 3. 스트림 파이프라인

### 중간 처리와 최종 처리

* 리덕션(Reduction)
  * 대량의 데이터를 가공해서 축소하는 것을 말한다.
    * 합계, 평균값, 카운팅, 최대값, 최소값등을 집계하는 것
  * 요소가 리덕션의 결과물로 바로 집계할 수 없을 경우 중간 처리가 필요하다.
    * 중간 처리 : 필터링, 매핑, 정렬, 그룹핑
  * 중간 처리한 요소를 최종 처리해서 리덕션 결과물을 산출한다.
* 스트림은 중간 처리와 최종 처리를 파이프라인으로 해결한다.
  * 파이프라인(pipelines): 스트림을 파이프처럼 이어 놓은 것을 말한다.
    * 중산처리 메소드(필터링, 매핑, 정렬)는 중간 처리된 스트림을 리턴하고,
    * 이 스트림에서 다시 중간 처리 메소드를 호출해서 파이프라인을 형성하게 된다.
  * **최종 스트림의 집계 기능이 시작되기 전까지 중간 처리는 지연(lazy)된다.**
    * 최종 스트림이 시작하면 비로소 컬렉션에서 요소가 하나씩 중간 스트림에서 처리되고 최종 스트림까지 오게된다.   
* 예) 회원 컬렉션에서 남자 회원의 평균 나이

```java
Stream<Member> maleFemaleStream = list.stream();
Stream<Member> maleStream = maleFemaleStream.filter(m -> m.getSex() == Member.MALE);
IntStream ageStream = maleStream.mapToInt(Member::getAge);
OptionalDouble optionalDouble = ageStream.average();
double ageAvg = optionalDouble.getAsDouble();
```

```java
double ageAvg = list.stream()				//오리지날 스트림
  .filter(m -> m.getSex() == Member.MALE)	//중간 처리 스트림
  .mapToInt(Member::getAge)					//중간 처리 스트림
  .average()								//최종 처리
  .getAsDouble();
```

* 예제 코드

```java
package sec03.stream_pipelines;

import java.util.Arrays;
import java.util.List;

public class StreamPipelinesEx {
  public static void main(String[] args) {
    List<Member> list = Arrays.asList(
        new Member("홍길동", Member.MALE, 30),
        new Member("김나리", Member.FEMALE, 20),
        new Member("김남준", Member.MALE, 25),
        new Member("박수미", Member.FEMALE, 28)
    );

    double avg = list.stream()
        .filter(m -> m.getSex() == Member.MALE)
        .mapToInt(Member::getAge)
        .average()
        .getAsDouble();
    System.out.println(avg);
  }
}
```

```java
남자 평균 나이: 27.5

Process finished with exit code 0
```

  

### 중간 처리 메소드롸 최종 처리 메소드

* 리턴 타입을 보면 중간 처리 메소드인지 최종 처리 메소드인지 구분할 수 있다.
  * 중간 처리 메소드 : 리턴 타입이 스트림
  * 최종 처리 메소드 : 리턴 타입이 기본 타입이거나 OptionalXXX
* 중간 처리 메소드
  * 중간 처리 메소드는 최종 처리 메소드가 실행되기 전까지 지연한다.
  * 최종 처리 메소드가 실행이 되어야만 동작을 한다.

![](https://github.com/namjunemy/TIL/blob/master/Java/img/16_1.png?raw=true)

* 최종 처리 메소드

![](https://github.com/namjunemy/TIL/blob/master/Java/img/16_2.png?raw=true)

  

## 4. 필터링 중간 처리 - distinct(), filter()

* 중간 처리 기능으로 요소를 걸러내는 역할을 한다.

![](https://github.com/namjunemy/TIL/blob/master/Java/img/16_3.png?raw=true)

* distinct()
  * 중복을 제거하는 스트림 필터링 메소드
  * **Stream:** equals() 메소드가 true가 나오면 동일한 객체로 판단하고 중복을 제거
  * **IntStream, LongStream, DoubleStream:** 동일값일 경우 중복을 제거
* filter()
  * 매개값으로 **주어진 Predicate가 true를 리턴하는 요소만 필터링**한다.

### distinct(), filter() 실습 예제

* distinct()로 List에 저장된 중복 객체를 제거하고,
* filter()를 통해서 조건에 맞는 객체만을 찾아서
* forEach()를 통해 하나씩 출력한다.

```java
package sec04.stream_filtering;

import java.util.Arrays;
import java.util.List;

public class FilteringEx {
  public static void main(String[] args) {
    List<String> names = Arrays.asList("홍길동", "신용권", "김자바", "신용권", "김남준", "김남준");

    names.stream()
        .distinct()
        .forEach(System.out::println);
    System.out.println();

    names.stream()
        .filter(str -> str.startsWith("신"))
        .forEach(System.out::println);
    System.out.println();

    names.stream()
        .distinct()
        .filter(str -> str.endsWith("준"))
        .forEach(System.out::println);
  }
}
```

```java
홍길동
신용권
김자바
김남준

신용권
신용권

김남준

Process finished with exit code 0
```

  

## 5. 매핑 중간 처리 - flatMapXXX(), mapXXX(), asXXXStream(), box

* 매핑(mapping)은 중간 처리 기능으로 스트림의 요소를 다른 요소로 대체 한다.

  * 예를 들면, 객체를 정수로, 객체를 double로, 객체를 boolean값으로 대체하는 것을 말한다. 반드시 하나의 요소가 하나의 요소로 대체 되는것은 아니다. 하나의 요소가 여러개의 요소로 대체될 수도 있다. 

* 매핑 메소드 종류

  * flatMapXXX(), mapXXX(), asDoubleStream(), asLongStream(), boxed()

  ​

### flatMapXXX()메소드

* 한 개의 요소를 대체하는 복수개의 요소들로 구성된 새로운 스트림을 리턴한다.

| 리턴타입         | 메소드(매개변수)                                | 요소 -> 대체 요소            |
| ------------ | ---------------------------------------- | ---------------------- |
| Stream\<R\>  | flatMap(Function\<T, Stream\<R\>\>)      | T -> Stream\<R\>       |
| DoubleStream | flatMap(DoubleFunction\<DoubleStream\>)  | double -> DoubleStream |
| IntStream    | flatMap(IntFunction\<IntStream\>)        | int -> IntStream       |
| LongStream   | flatMap(LongFunction\<LongStream\>)      | long -> longStream     |
| DoubleStream | flatMapToDouble(Function\<T, DoubleStream\>) | T -> DoubleStream      |
| IntStream    | flatMapToInt(Function\<T, IntStream\>)   | T -> IntStream         |
| LongStream   | flatMapToLong(Function\<T, LongStream\>) | T -> LingStream        |

* **flatMapXXX() 실습 예제**
  * 다음 예제는 입력된 데이터(요소)들이 List\<String\>에 저장되어 있다고 가정하고, 요소별로 단어를 뽑아 단어 스트림으로 재생성한다. 만약 입력된 데이터들이 숫자라면 숫자를 뽑아 숫자 스트림으로 재생성한다.

```java
package sec05.stream_mapping;

import java.util.Arrays;
import java.util.List;

public class FlatMapEx {
  public static void main(String[] args) {
    List<String> inputList1 = Arrays.asList("java8 lambda", "stream mapping");

    inputList1.stream()
        .flatMap(data -> Arrays.stream(data.split(" ")))
        .forEach(System.out::println);

    System.out.println();

    List<String> inputList2 = Arrays.asList("10, 20, 30", "40, 50, 60");
    inputList2.stream()
        .flatMapToInt(data -> {
          String[] strArray = data.split(",");
          int[] intArray = new int[strArray.length];
          for (int i = 0; i < strArray.length; i++) {
            intArray[i] = Integer.parseInt(strArray[i].trim());
          }
          return Arrays.stream(intArray);
        })
        .forEach(System.out::println);
  }
}
```

```java
java8
lambda
stream
mapping

10
20
30
40
50
60

Process finished with exit code 0
```

  

### mapXXX() 메소드

* 요소를 대체하는 요소로 구성된 새로운 스트림을 리턴한다.

  * flatMapXXX() 메소드는 하나의 요소를 여러개의 요소로 대체 하고, mapXXX() 메소드는 하나의 요소로 대체 한다.

  ![](https://github.com/namjunemy/TIL/blob/master/Java/img/16_4.png?raw=true)

* **mapXXX() 실습 예제**

```java
package sec05.stream_mapping;

import java.util.Arrays;
import java.util.List;

public class MapEx {
  public static void main(String[] args) {
    List<Student> studentList = Arrays.asList(
        new Student("홍길동", 10),
        new Student("신용권", 20),
        new Student("김남준", 30)
    );

    studentList.stream()
        .mapToInt(Student::getScore)
        .forEach(System.out::println);

  }
}
```

```java
10
20
30

Process finished with exit code 0
```

  

### asDoubleStream(), asLongStream(), boxed() 메소드

| 리턴타입                                     | 메소드(매개변수)        | 설명                                       |
| ---------------------------------------- | ---------------- | ---------------------------------------- |
| DoubleStream                             | asDoubleStream() | int -> double, long -> double            |
| LongStream                               | asLongStream()   | int -> long                              |
| Stream\<Integer\>, Stream\<Long\>, Stream\<Double\> | boxed()          | int -> Integer, long -> Long, double -> Double |

* asDoubleStream()
  * IntStream()의 int요소 또는 LongStream의 long요소를 double요소로 타입 변환해서 DoubleStream을 생서
* asLongStream()
  * IntStream()의 int요소를 long요소로 타입 변환해서 LongStream을 생성
* boxed()
  * int요소, long요소, double요소를 Integer, Long, Double요소로 박싱해서 Stream을 생성
* **asDoubleStream, boxed() 실습 예제**

```java
package sec05.stream_mapping;

import java.util.Arrays;
import java.util.stream.IntStream;

public class AsDoubleStreamAndBoxedEx {
  public static void main(String[] args) {
    int[] intArray = {1, 2, 3, 4, 5};

    IntStream intStream = Arrays.stream(intArray);
    intStream.asDoubleStream().forEach(System.out::println);

    System.out.println();

    intStream = Arrays.stream(intArray);
    intStream.boxed().forEach(System.out::println);
  }
}
```

```java
1.0
2.0
3.0
4.0
5.0

1
2
3
4
5

Process finished with exit code 0
```

  

## 6. 정렬 중간 처리 - sorted()

* 중간 처리 기능으로 최종 처리되기 전에 요소를 정렬한다.

| 리턴타입         | 메소드(매개변수)               | 설명                         |
| ------------ | ----------------------- | -------------------------- |
| Stream\<T\>  | sorted()                | 객체를 Comarable 구현 방법에 따라 정렬 |
| Stream\<T\>  | sorted(Comparator\<T\>) | 객체를 주어진 Comparator에 따라 정렬  |
| DoubleStream | sorted()                | double 요소를 올림 차순으로 정렬      |
| IntStream    | sorted()                | int 요소를 올림 차순으로 정렬         |
| LongStream   | sorted()                | long 요소를 올림 차순으로 정렬        |

* 객체 요소일 경우에는 Comparable을 구현하지 않으면 첫번째 sorted() 메소드를 호출하면 ClassCastException이 발생

* 객체 요소가 Compatable을 구현하지 않았거나, 구현 했다하더라도 다른 비교 방법으로 정렬하려면 Comparator를 매개값으로 갖는 두번째 sorted() 메소드를 사용해야 한다.

  * Comparator는 함수적 인터페이스이므로 다음과 같이 람다식으로 매개값을 작성할 수 있다.

    ```java
    sorted((a, b) -? { ... })
    ```

  * 중괄호 {} 안에는 a와 b를 비교해서 a가 작으면 음수, 같으면 0, a가 크면 양수를 리턴하는 코드를 작성

* 정렬 코드 예

  * 객체 요소가 Comparable을 구현하고 있고, 기본 비교(Comparable) 방법으로 정렬

  * 다음 세가지 방법 중 하나를 선택

    ```java
    sorted();
    sorted((a, b) -> a.compareTo(b));
    sorted(Comparator.naturalOrder());
    ```

  * 요소가 Comparable을 구현하고 있지만, 기본 비교 방법과 정반대 방법으로 정렬

    * 위에서의 정렬 방법과 정반대의 결과가 나온다.

    ```java
    sorted((a, b) -> b.compareTo(a));
    sorted(Comparator.reverseOrder());
    ```

### 정렬 실습 예제

* sorted() 메소드를 이용하여 스트림 중간 처리를 한다. Comparator.reverseOrder()를 통해 정반대의 정렬을 할 수 있다.

```java
package sec06.stream_sorting;

public class Student implements Comparable<Student> {
  private String name;
  private int score;

  public Student(String name, int score) {
    this.name = name;
    this.score = score;
  }

  public String getName() {
    return name;
  }

  public int getScore() {
    return score;
  }

  @Override
  public int compareTo(Student o) {
    return Integer.compare(score, o.score);
  }
}
```

```java
package sec06.stream_sorting;

import java.util.Arrays;
import java.util.Comparator;
import java.util.List;
import java.util.stream.IntStream;

public class SortingEx {
  public static void main(String[] args) {
    IntStream intStream = Arrays.stream(new int[]{5, 3, 2, 1, 4,});

    intStream.sorted().forEach(n -> System.out.print(n + ","));
    System.out.println();

    List<Student> studentList = Arrays.asList(
        new Student("홍길동", 30),
        new Student("신용권", 33),
        new Student("김남준", 25)
    );

    studentList.stream()
        .sorted()
        .forEach(student -> System.out.print(student.getScore() + ","));

    System.out.println();

    studentList.stream()
        .sorted(Comparator.reverseOrder())
        .forEach(student -> System.out.print(student.getScore() + ","));

  }
}
```

```java
1,2,3,4,5,
25,30,33,
33,30,25,
Process finished with exit code 0
```

  

## 7. 루핑(looping)

* 중간 또는 최종 처리 기능으로 요소 전체를 반복하는 것을 말한다.

* 루핑 메소드

  * peek() : 중간 처리 메소드

    * 최종 처리 메소드가 실행되지 않으면 지연되기 때문에 최종 처리 메소드가 호출되어야만 동작한다.

    * 루핑 미동작

      ```java
      intStream
        .filter(a -> a % 2 == 0)
        .peek(System.out::println);
      ```

    * 루핑 동작

      ```java
      intStream
        .filter(a -> a % 2 == 0)
        .peek(System.out::println)
        .sum()
      ```

  * forEach() : 최종 처리 메소드

    ```java
    intStream
      .filter(a -> a % 2 == 0)
      .forEach(System.out::println);
    ```

### 루핑 실습 예제

```java
package sec07.stream_looping;

import java.util.Arrays;

public class LoopingEx {
  public static void main(String[] args) {
    int[] intArr = {1, 2, 3, 4, 5};
    System.out.println("[peek()를 마지막에 호출한 경우]");
    Arrays.stream(intArr)
        .filter(a -> a % 2 == 0)
        .peek(System.out::println); //동작하지 않는다.
    System.out.println();

    System.out.println("[최종 처리 메소드를 마지막에 호출한 경우]");
    int sum = Arrays.stream(intArr)
        .filter(a -> a % 2 == 0)
        .peek(System.out::println)
        .sum();
    System.out.println("총합: " + sum);
    
    System.out.println("[forEach()를 마지막에 호출한 경우]");
    Arrays.stream(intArr)
        .filter(a -> a % 2 == 0)
        .forEach(System.out::println);
  }
}
```

```java
[peek()를 마지막에 호출한 경우]

[최종 처리 메소드를 마지막에 호출한 경우]
2
4
총합: 6
[forEach()를 마지막에 호출한 경우]
2
4

Process finished with exit code 0
```

  

## 8. 매칭(matching) 최종 처리

* 최종 처리 기능으로 요소들이 특정 조건을 만족하는지 조사하는 것을 말한다.

* 매칭 메소드

  * allMatch()
    * 모든 요소들이 매개값으로 주어진 Predicate의 조건을 만족하는지 조사
  * anyMatch()
    * 최소한 한 개의 요소가 매개값으로 주어진 Predicate의 조건을 만족하는지 조사
  * noneMatch()
    * 모든 요소들이 매개값으로 주어진 Predicate의 조건을 만족하지 않는지 조사

  ![](https://github.com/namjunemy/TIL/blob/master/Java/img/16_5.png?raw=true)

### 매칭 실습 예제

```java
package sec08.stream_match;

import java.util.Arrays;

public class MatchEx {
  public static void main(String[] args) {
    int[] intArray = {2, 4, 6};

    boolean result = Arrays.stream(intArray).allMatch(a -> a % 2 == 0);
    System.out.println("모두 2의 배수인가? " + result);

    result = Arrays.stream(intArray).anyMatch(a -> a % 3 == 0);
    System.out.println("3의 배수가 있는가? " + result);

    result = Arrays.stream(intArray).noneMatch(a -> a % 3 == 0);
    System.out.println("3의 배수가 없는가? " + result);
  }
}
```

```java
모두 2의 배수인가? true
3의 배수가 있는가? true
3의 배수가 없는가? false

Process finished with exit code 0
```

  

## 9. 기본 집계 최종 처리

### 집계(Aggregate)

* 최종 처리 기능
  * 카운팅, 합계, 평균값, 최대값, 최소값등과 같이 하나의 값으로 산출한다.
  * 대량의 데이터를 가공해서 축소하는 리덕션(Reduction)이라고 볼 수 있다.
* 스트림이 제공하는 기본 집계 함수

| 리턴타입              | 메소드(매개변수)            | 설명     |
| ----------------- | -------------------- | ------ |
| long              | count()              | 요소 개수  |
| OptionalXXX       | findFirst()          | 첫번째 요소 |
| Optional\<T\>     | max(Compatator\<T\>) | 최대 요소  |
| OptionalXXX       | max()                | 최대 요소  |
| Optional\<T\>     | min(Comparator\<T\>) | 최소 요소  |
| OptionalXXX       | min()                | 최소 요소  |
| OptionalDouble    | average()            | 요소 평균  |
| int, long, double | sum()                | 요소 총합  |

* OptionalXXX 클래스
  * 자바8부터 추가된 값을 저장하는 값 기반 클래스
  * java.util 패키지의 Optional, OptionalDouble, OptionalInt, OptionalLong 클래스를 말한다.
    * 저장괸 값을 얻으려면 get(), getAsDouble(), getAsInt(), getAsLong() 메소드를 호출한다.

### Aggregate 실습 예제

```java
package sec09.stream_aggregate;

import java.util.Arrays;

public class AggregateEx {
  public static void main(String[] args) {
    long count = Arrays.stream(new int[]{1, 2, 3, 4, 5})
        .filter(n -> n % 2 == 0)
        .count();
    System.out.println("2의 배수의 수: " + count);

    long sum = Arrays.stream(new int[]{1, 2, 3, 4, 5})
        .filter(n -> n % 2 == 0)
        .sum();
    System.out.println("2의 배수의 합: " + sum);

    int max = Arrays.stream(new int[]{1, 2, 3, 4, 5})
        .filter(n -> n % 2 == 0)
        .max()
        .getAsInt();
    System.out.println("2의 배수 중 최대값: " + max);

    int min = Arrays.stream(new int[]{1, 2, 3, 4, 5})
        .filter(n -> n % 2 == 0)
        .min()
        .getAsInt();
    System.out.println("2의 배수 중 최소값: " + min);

    int first = Arrays.stream(new int[]{1, 2, 3, 4, 5})
        .filter(n -> n % 3 == 0)
        .findFirst()
        .getAsInt();
    System.out.println("첫번째 3의 배수: " + first);
  }
}
```

```java
2의 배수의 수: 2
2의 배수의 합: 6
2의 배수 중 최대값: 4
2의 배수 중 최소값: 2
첫번째 3의 배수: 3

Process finished with exit code 0
```

  

### Optional 클래스

* 값을 저장하는 값 기반 클래스
  * Optional, OptionalDouble, OptionalInt, OptionalLong
  * 집계 메소드의 리턴 타입으로 사용되어 집계 값을 가지고 있늠
* 특징
  * 집계 값이 존재하지 않을 경우 디폴트 값을 설정할 수도 있다.
  * 집계 값을 처리하는 Consumer를  등록할 수 있다.

| 리턴타입    | 메소드(매개변수)                 | 설명                          |
| ------- | ------------------------- | --------------------------- |
| boolesn | isPresent()               | 값이 저장되어 있는지 여부              |
| T       | orElse(T)                 | 값이 저장되어 있지 않을 경우 디폴트 값      |
| double  | orElse(double)            | 값이 저장되어 있지 않을 경우 디폴트 값      |
| int     | orElse(int)               | 값이 저장되어 있지 않을 경우 디폴트 값      |
| long    | orElse(long)              | 값이 저장되어 있지 않을 경우 디폴트 값      |
| void    | ifPresent(Consumer)       | 값이 저장되어 있을 경우 Consumer에서 처리 |
| void    | ifPresent(DoubleConsumer) | 값이 저장되어 있을 경우 Consumer에서 처리 |
| void    | ifPresent(IntConsumer)    | 값이 저장되어 있을 경우 Consumer에서 처리 |
| void    | ifPresent(LongConsumer)   | 값이 저장되어 있을 경우 Consumer에서 처리 |

### Optional 실습 예제

```java
package sec09.stream_aggregate;

import java.util.ArrayList;
import java.util.List;
import java.util.OptionalDouble;

public class OptionalEx {
  public static void main(String[] args) {
    List<Integer> list = new ArrayList<>();

    OptionalDouble optional = list.stream()
        .mapToInt(Integer::intValue)
        .average();
    if(optional.isPresent()) {
      System.out.println("방법1 평균: " + optional.getAsDouble());
    } else {
      System.out.println("방법1 평균: 0.0");
    }

    double avg = list.stream()
        .mapToInt(Integer::intValue)
        .average()
        .orElse(0.0);
    System.out.println("orElse를 이용한 평균: " + avg);

    list.add(2);
    list.add(4);
    list.stream()
        .mapToInt(Integer::intValue)
        .average()
        .ifPresent(a -> System.out.println("isPresent를 이용한 평균: " + a));
  }
}
```

```java
방법1 평균: 0.0
orElse를 이용한 평균: 0.0
isPresent를 이용한 평균: 3.0

Process finished with exit code 0
```

  

## 커스텀 집계 - reduce()

### reduce() 메소드

* 프로그램화해서 다양한 집계(리덕션) 결과물을 만들수 있다.
  * Stream 인터페이스 타입의 두 메소드를 보면
  * 첫번째 메소드는 BinaryOperator를 매개 변수로 받으므로 두 값의 연산 결과를 Optional하게 리턴한다는 의미이고
  * 두번째 메소드는 두 값의 연산결과를 리턴하되, 결과가 없다면(연산이 안되는 경우 즉, 두개의 요소가 없는 예외 발생 경우 NoSuchElementException) default로 identity값을 리턴한다는 의미이다.

| 인터페이스        | 리턴타입           | 메소드(매개변수)                                |
| ------------ | -------------- | ---------------------------------------- |
| Stream       | Optional\<T\>  | reduce(BinaryOperator\<T\> accumulator)  |
| Stream       | T              | reduce(T identity, BinaryOperator\<T\> accumulator) |
| IntStream    | OptionalInt    | reduce(IntBinaryOperator op)             |
| IntStream    | int            | reduce(int identity, IntBinaryOperator op) |
| LongStream   | OptionalLong   | reduce(LongBinaryOperator op)            |
| LongStream   | long           | reduce(long identity, LongBinaryOperator op) |
| DoubleStream | OptionalDouble | reduce(DoubleBinaryOperator op)          |
| DoubleStream | double         | reduce(double identity, DoubleBinaryOperator op) |

* 매개변수
  * XXXBinaryOperator : 두 개의 매개값을 받아 연산 후 리턴하는 함수적 인터페이스
  * identity : 스트림에 요소가 전혀 없을 경우 리턴될 디폴트 값
* 예: 학생들의 성적 총점

```java
int sum = studentList.stream()
  .map(Student::getScore)
  .reduce((a, b) -> a + b)
  .get();
```

```java
int sum = studentList.stream()
  .map(Student::getScore)
  .reduce(0, (a, b) -> a + b)
```

### reduce() 실습 예제

```java
package sec10.stream_reduce;

import java.util.Arrays;
import java.util.List;

public class ReductionEx {
  public static void main(String[] args) {
    List<Student> studentList = Arrays.asList(
        new Student("홍길동", 90),
        new Student("신용권", 70),
        new Student("김남준", 98)
    );

    int sum1 = studentList.stream()
        .mapToInt(Student::getScore)
        .sum();

    int sum2 = studentList.stream()
        .mapToInt(Student::getScore)
        .reduce((a, b) -> a + b)
        .getAsInt();

    int sum3 = studentList.stream()
        .mapToInt(Student::getScore)
        .reduce(0, (a, b) -> a + b);

    System.out.println("sum1: " + sum1);
    System.out.println("sum2: " + sum2);
    System.out.println("sum3: " + sum3);
  }
}
```

```java
sum1: 258
sum2: 258
sum3: 258

Process finished with exit code 0
```

  

## 11. 수집 최종 처리 - collect()

### collect()

최종 처리 기능으로 요소들을 수집 또는 그룹핑한다.

* 필터링 또는 매핑된 요소들로 구성된 새로운 컬렉션을 생성한다.
* 요소들을 그룹핑하고, 집계(리덕션)을 할 수 있다.
  * 예를 들면, 전체 학생 요소들 중에서 남학생과 여학생을 따로 그룹핑하여 집계할 수 있다.

### 필터링한 요소 수집

| 리턴타입 | 메소드(매개변수)                               | 인터페이스  |
| ---- | --------------------------------------- | ------ |
| R    | collect(Collector\<T, A, R\> collector) | Stream |

* Collector의 타입 파라미터
  * T : 요소
  * A : 누적기(accumulator)
  * R : 요소가 저장될 새로운 컬렉션
  * ==> T요소를 A누적기가 R에 저장한다.