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

