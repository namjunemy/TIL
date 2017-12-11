# 15장. 컬렉션

> [소스코드 repo](https://github.com/namjunemy/this_is_java)
>
> 1절. 컬렉션 프레임워크 소개
>
> 2절. List 컬렉션
>
> 3절. Set 컬렉션
>
> 4절. Map 컬렉션
>
> 5절. 검색 기능을 강화시킨 컬렉션
>
> 6절. LIFO와 FIFO 컬렉션
>
> 7절. 동기화된(synchronized) 컬렉션
>
> 8절. 병렬 처리(Concurrent)를 위한 컬렉션

  

## 1. 컬렉션 프레임워크 소개

### 컬렉션 프레임워크(Collection Framework)

* 컬렉션
  * 사전적 의미로 요소(객체)를 수집해서 저장하는 것
* 배열의 문제점
  * 저장할 수 있는 객체 수가 배열을 생성할 때 결정
    * 불특정 다수의 객체를 저장하기에는 문제
  * 객체를 삭제했을 때 해당 인덱스가 비게됨
    * 낱알이 군데군데 빠진 옥수수가 될 수 있다.
    * 객체를 저장하려면 어디가 비어있는지 확인해야한다.
* 컬렉션 프레임워크
  * 객체들을 효율적으로 추가, 삭제, 검색할 수 있도록 제공되는 컬렉션 라이브러리
  * java.util 패키지에 포함
  * 인터페이스를 통해서 정형화된 방법으로 다양한 컬렉션 클래스를 이용

### 컬렉션 프렘워크의 주요 인터페이스

* List
  * 배열과 유사하게 인덱스로 관리
* Set
  * 집합과 유사. 구슬 주머니에 객체를 저장하는 것과 같은 형태
* Map
  * 키와 값의 쌍으로 관리

| 인터페이스 분류   | 분류      | 특징                        | 구현 클래스                                  |
| ---------- | ------- | ------------------------- | --------------------------------------- |
| Collection | List 계열 | 순서를 유지하고  저장, 중복 저장 가능    | ArrayList, Vector, LinkedList           |
| Collection | Set 계열  | 순서를 유지하지 않고 저장, 중복 저장 불가  | HashSet, TreeSet                        |
|            | Map 계열  | 키와 값의 쌍으로 저장, 키는 중복 저장 안됨 | HashMap, Hashtable, TreeMap, Properties |

  

## 2. List 컬렉션

### List 컬렉션의 특징 및 주요 메소드

* 특징
  * 인덱스로 관리
  * 중복해서 객체 저장 가능
* 구현 클래스
  * ArrayList
  * Vector
  * LinkedList
* 주요 메소드

| 기능    | 메소드                            | 설명                          |
| ----- | ------------------------------ | --------------------------- |
| 객체 추가 | boolean add(E e)               | 주어진 객체를 맨끝에 추가              |
|       | void add(int index, E element) | 주어진 인덱스에 객체를 추가             |
|       | set(int index, E element)      | 주어진 인덱스에 저장된 객체를 주어진 객체로 바꿈 |
| 객체 검색 | boolean contains(Object o)     | 주어진 인덱스에 저장된 객체를 리턴         |
|       | E get(int index)               | 주어진 인덱스에 저장된 객체를 리턴         |
|       | isEmpty()                      | 컬렉션이 비어 있는지 조사              |
|       | int size()                     | 저장되어있는 전체 객체수를 리턴           |
| 객체 삭제 | void clear()                   | 주어진 인덱스에 저장된 객체를 삭제         |
|       | E remove(int index)            | 주어진 인덱스에 저장된 객체를 삭제 후 리턴    |
|       | boolean remove(Object o)       | 주어진 객체를 삭제                  |

* 전체 객체를 대상으로 반복해서 얻기
  * 저장된 총 객체수 만큼 루핑

```java
List<String> list = new ArrayList<>;
for(int i = 0; i < list.size(); i++) {
  String str = list.get(i);
}

또는

for(String str : list) {
  ...
}
```

  

### ArrayList

* 저장 용량(capacity)

  * 초기 : 10
  * 초기 용량 지정 가능
  * 저장 용량을 초과한 객체들이 들어오면 자동적으로 늘어난다.

* 객체 제거

  * 바로 뒤 인덱스부터 마지막 인덱스까지 모두 앞으로 1씩 당겨진다.

* 고정된 객체들로 구성된 List 생성

  * `List<T> list = Arrays.asList(T... a);`
  * 자동언박싱

  ```java
  List<String> list1 = Arrays.asList("홍길동", "김남준", "김자바");
  for(String name : list1) {
    System.out.println(name);
  }

  List<Integer> list2 = Arrays.asList(1, 2, 3);
  for(int value : list2) {
    System.out.println(value);
  }
  ```

*  예제

```java
package list;

import java.util.ArrayList;
import java.util.List;

public class ArrayListEx {
  public static void main(String[] args) {
    List<String> list = new ArrayList<>();

    list.add("Java");
    list.add("JDBC");
    list.add("Servlet/JSP");
    list.add(2, "Database");
    list.add("Mybatis");
    System.out.println(list.size());
    System.out.println(list);

    String skill = list.get(2);
    System.out.println("2: " + skill);

    list.forEach(str -> System.out.println(list.indexOf(str) + ":" + str + " "));
  }
}
```

```java
5
[Java, JDBC, Database, Servlet/JSP, Mybatis]
2: Database
0:Java 
1:JDBC 
2:Database 
3:Servlet/JSP 
4:Mybatis 

Process finished with exit code 0
```



### Vector

```java
List<E> list = new Vector<>();
```

* 특징
  * Vector는 스레드 동기화(synchronization)가 되어 있기 때문에 복수의 스레드가 동시에 Vector에 접근해서 객체를 추가, 삭제 하더라도 스레드에 안전 하다.(Thread Safe 하다.)
  * 그렇기 때문에 멀티스레드 환경에서 index로 데이터를 관리하고 싶은 경우 Vector를 사용하는 것이 안전하다.
  * 반면에, 싱글스레드 환경에서는 ArrayList를 사용하는 것이 더 빠르다. synchronized처리가 되어있지 않기 때문이다.
* 예제

```java
package list.ex2_vector;

public class Board {
  private String subject;
  private String content;
  private String writer;

  public Board(String subject, String content, String writer) {
    this.subject = subject;
    this.content = content;
    this.writer = writer;
  }

  @Override
  public String toString() {
    return subject + " " + content + " " + writer;
  }
}
```

```java
package list.ex2_vector;

import java.util.List;
import java.util.Vector;

public class VectorEx {
  public static void main(String[] args) {
    List<Board> list = new Vector<>();

    list.add(new Board("제목1", "내용1", "작성자1"));
    list.add(new Board("제목2", "내용2", "작성자2"));
    list.add(new Board("제목3", "내용3", "작성자3"));
    list.add(new Board("제목4", "내용4", "작성자4"));
    list.add(new Board("제목5", "내용5", "작성자5"));

    list.remove(2);
    list.remove(3);

    list.forEach(board -> System.out.println(board));
  }
}

```

```java
제목1 내용1 작성자1
제목2 내용2 작성자2
제목4 내용4 작성자4
```



### LinkedList

```java
List<E> list = new LinkedList<>();
```

* 특징
  * 인접 참조를 링크해서 체인처럼 관리
  * 특정 인덱스에서 객체를 제거하거나 추가하게 되면 바로 앞뒤 링크만 변경
  * 빈번한 객체 삭제와 삽입이 일어나는 곳에서는 ArrayList보다 좋은 성능을 발휘(특정 인덱스일 경우 훨씬 더)
* 예제

```java
package list.ex3_linkedlist;

import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

public class ArrayLinked {
  public static void main(String[] args) {
    List<String> arrayList = new ArrayList<>();
    List<String> linkedList = new LinkedList<>();

    long startTime;
    long endTime;

    startTime = System.nanoTime();
    for (int i = 0; i < 10000; i++) {
      arrayList.add(0, String.valueOf(i));
    }
    endTime = System.nanoTime();
    System.out.println("arrayList 걸린 시간 : " + (endTime - startTime) + "ns");

    startTime = System.nanoTime();
    for (int i = 0; i < 10000; i++) {
      linkedList.add(0, String.valueOf(i));
    }
    endTime = System.nanoTime();
    System.out.println("linkedList 걸린 시간 : " + (endTime - startTime) + "ns");
  }
}
```

```java
arrayList 걸린 시간 : 17661954ns
linkedList 걸린 시간 : 4967602ns

Process finished with exit code 0
```

