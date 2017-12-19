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

  ​