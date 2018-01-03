# 15장. 컬렉션

> '이것이 자바다 - 신용권' 15장 학습
>
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

### 컬렉션 프레임워크의 주요 인터페이스

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

  

## 3. Set 컬렉션

### Set 컬렉션의 특징 및 주요 메소드

* 특징
  * 수학의 집합에 비유될 수 있다.
  * 저장 순서가 유지되지 않는다.
  * 객체를 중복해서 저장할 수 없고,
  * 하나의 null만 저장할 수 있다.
* 구현 클래스
  * HashSet, LinkedHashSet, TreeSet
* 주요 메소드

| 기능    | 메소드                        | 설명                                       |
| ----- | -------------------------- | ---------------------------------------- |
| 객체 추가 | boolean add(E e)           | 주어진 객체를 저장, 객체가 성공적으로 저장되면 true를 리턴하고 중복 객체면 false를 리턴 |
| 객체 검색 | boolean contains(Object o) | 주어진 객체가 저장되어 있는지 여부                      |
|       | isEmpty()                  | 컬렉션이 비어 있는지 조사                           |
|       | Iterator\<E\> iterator()   | 저장된 객체를 한번씩 가져오는 반복자 리턴                  |
|       | int size()                 | 저장되어있는 전체 객체 수 리턴                        |
| 객체 삭제 | void clear()               | 저장된 모든 객체를 삭제                            |
|       | boolean remove(Object o)   | 주어진 객체를 삭제                               |

* 객체 추가 및 삭제

```java
Set<String> set = ...;
set.add("홍길동");
set.remove("홍길동");
```

* Set 컬렉션은 인덱스로 객체를 검색해서 가져오는 메소드가 없다.
* 대신, 전체 객체를 대상으로 한번씩 반복해서 가져오는 **반복자(Iterator)**를 제공한다.

```java
Set<String> set = ...;
Iterator<String> iterator = set.iterator();
```

| 리턴타입    | 메소드명      | 설명                                      |
| ------- | --------- | --------------------------------------- |
| boolean | hasNext() | 가져올 객체가 있으면 true를 반환하고 없으면 false를 반환한다. |
| E       | next()    | 컬렉션에서 하나의 객체를 가져온다.                     |
| void    | remove()  | Set 컬렉션에서 객체를 제거한다.                     |

```java
Set<String> set = ...;
Iterator<String> iterator = set.iterator();
while(iterator.hasNext()) {
  String str = iterator.next();
}

or

for(String str : set) {
  ...
}
```

* 반복자를 통한 객체 제거
  * iterator의 메소드이지만 set컬렉션의 객체가 제거된다.

```java
while(iterator.hasNext()) {
  String str = iterator.next();
  if(str.equals("홍길동")) {
    iterator.remove();
  }
}
```

  

### HashSet

```java
Set<E> set = new HashSet<E>();
```

* 특징

  * 동일 객체 및 동등 객체는 중복 저장하지 않는다.
  * 동등 객체 판단 방법
    * 동등객체가 되려면?
      * 두 객체의 hashCode()의 리턴값이 같을 경우, 다시 두 객체의 equals()의 리턴값을 비교하여 같을 경우 동등 객체로 판단한다.
      * hashCode() 리턴값이 다르거나, hashCode() 리턴값은 같지만 equals()의 리턴값이 다른 경우 다른 객체로 판단한다.

* 예제

  * HashSet

  ```java
  package set;

  import java.util.HashSet;
  import java.util.Iterator;
  import java.util.Set;

  public class HashSetEx {
    public static void main(String[] args) {
      Set<String> set = new HashSet<>();

      set.add("java");
      set.add("JDBC");
      set.add("java");
      set.add("Servlet/JSP");
      set.add("mybatis");

      System.out.println(set.size());

      Iterator<String> iterator = set.iterator();
      while (iterator.hasNext()) {
        System.out.print(iterator.next() + " ");
      }

      System.out.println();
      for (String str : set) {
        System.out.print(str + " ");
      }

      System.out.println();
      set.clear();
      System.out.println(set);
    }
  }
  ```

  ```java
  4
  java mybatis JDBC Servlet/JSP 
  java mybatis JDBC Servlet/JSP 
  []

  Process finished with exit code 0
  ```

  * hashCode(), equals() Override

  ```java
  package set.hashcode_equals;

  import java.util.Objects;

  public class Member {
    public String name;
    public int age;

    public Member(String name, int age) {
      this.name = name;
      this.age = age;
    }

    @Override
    public boolean equals(Object object) {
      if(object instanceof Member) {
        Member member = (Member) object;
        return member.name.equals(name) && member.age == age;
      } else {
        return false;
      }
    }

    @Override
    public int hashCode() {
      return Objects.hash(name, age);
    }
  }
  ```

  ```java
  package set.hashcode_equals;

  import java.util.HashSet;
  import java.util.Set;

  public class HashSetEx2 {
    public static void main(String[] args) {
      Set<Member> set = new HashSet<>();

      set.add(new Member("홍길동", 30));
      set.add(new Member("홍길동", 30));

      System.out.println(set.size());
    }
  }
  ```

  ```java
  1

  Process finished with exit code 0
  ```





## 4. Map 컬렉션

### Map 컬렉션의 특징 및 주요 메소드

- 특징
  - 키(Key)와 값(value)으로 구성된 Map.Entry 객체를 저장하는 구조
  - 키와 값은 모두 객체
  - 키는 중복될 수 없지만 값은 중복 저장 가능
- 구현 클래스
  - HashMap, Hashtable, LinkedHashMap, Properties, TreeMap
- 주요 메소드

| 기능    | 메소드                                 | 설명                                       |
| ----- | ----------------------------------- | ---------------------------------------- |
| 객체 추가 | V put(K key, V value)               | 주어진 키와 값을 추가, 저장이 되면 값을 리턴               |
| 객체 검색 | boolean containsKey(Object key)     | 주어진 키가 있는지 여부                            |
|       | boolean containsValue(Object value) | 주어진 값이 있는지 여부                            |
|       | Set\<Map.Entry\<K,V\>\> entrySet()  | 키와 값의 쌍으로 구성된 모든 Map.Entry 객체를 Set에 담아서 리턴 |
|       | V get(Object key)                   | 주어진 키의 값을 리턴                             |
|       | boolean isEmpty()                   | 컬렉션이 비어있는지 여부                            |
|       | Set\<K\> keySet()                   | 모든 키를 Set 객체에 담아서 리턴                     |
|       | int size()                          | 저장된 키의 총 수를 리턴                           |
|       | Collection \<V\> values()           | 저장된 모든 값 Collection에 담아서 리턴              |
| 객체 삭제 | void clear()                        | 모든 Map.Entry(키와 값)를 삭제                   |
|       | V remove(Object key)                | 주어진 키와 일치하는 Map.Entry 삭제, 삭제가 되면 값을 리턴   |

* 객체 추가, 찾기, 삭제

```java
Map<String, Integer> map = ...;
map.put("홍길동", 30);
int score = map.get("홍길동");
map.remove("홍길동")
```

* 전체 객체를 대상으로 반복해서 얻기
  * Key-Value 쌍을 반복적으로 다루기 위해서 Set인터페이스 형태로 keySet과 entrySet을 저장하고 iterator를 이용해 반복 접근한다.

```java
Map<K,V> map = ...;
Set<K> keySet = map.keySet();
Iterator<K> keyIterator = keySet.iterator();
while(keyIterator.hasNext()) {
  K key = keyIterator.next();
  V value = map.get(key);
}
```

```java
Set<Map.Entry<K,V>> entrySet = map.entrySet();
Iterator<Map.Entry<K,V>> entryIterator = entrySet.iterator();
while(entryIterator.hasNext()) {
  Map.Entry<K,V> entry = entryIterator.next();
  K key = entry.getKey();
  V value = entry.getValue();
}
```

  

### HashMap

```java
Map<K, V> map = new HashMap<K, V>();
```

* 특징

  * 키 객체는 hashCode()와 equals()를 재정의해서 동등객체가 될 조건을 정해야 한다.
    * 동등객체가 되려면?
      - 두 객체의 hashCode()의 리턴값이 같을 경우, 다시 두 객체의 equals()의 리턴값을 비교하여 같을 경우 동등 객체로 판단한다.
      - hashCode() 리턴값이 다르거나, hashCode() 리턴값은 같지만 equals()의 리턴값이 다른 경우 다른 객체로 판단한다.
  * 키 타입은 String을 많이 사용
    * String은 문자열이 같을 경우 동등 객체가 될 수 있도록 hashCode()와 equals()메소드가 재정의되어 있기 때문

* 예제

  * HaspMap Iterator 사용

  ```java
  package map.hashmap;

  import java.util.HashMap;
  import java.util.Iterator;
  import java.util.Map;
  import java.util.Set;

  public class HaspMapEx1 {
    public static void main(String[] args) {
      Map<String, Integer> map = new HashMap<>();

      map.put("신용권", 40);
      map.put("홍길동", 30);
      map.put("김남준", 32);
      map.put("홍길동", 32);
      System.out.println(map.size());
      System.out.println(map);

      Set<Map.Entry<String, Integer>> entrySet = map.entrySet();
      Iterator<Map.Entry<String, Integer>> entryIterator = entrySet.iterator();

      System.out.println("##entrySet()");
      while(entryIterator.hasNext()) {
        Map.Entry<String, Integer> entry = entryIterator.next();
        System.out.print(entry.getKey() + " ");
        System.out.println(entry.getValue());
      }

      Set<String> keySet = map.keySet();
      Iterator<String> keyIterator = keySet.iterator();

      System.out.println("##keySet()");
      while(keyIterator.hasNext()) {
        String key = keyIterator.next();
        int value = map.get(key);
        System.out.println(key + " " + value);
      }

      map.clear();
      System.out.println(map.size());
    }
  }
  ```

  ```java
  3
  {홍길동=32, 신용권=40, 김남준=32}
  ##entrySet()
  홍길동 32
  신용권 40
  김남준 32
  ##keySet()
  홍길동 32
  신용권 40
  김남준 32
  0

  Process finished with exit code 0
  ```

  * 중복 키값 저장 안함, 가장 최근 추가된 value로 대체

  ```java
  package map.hashcode_equals;

  import java.util.HashMap;
  import java.util.Map;

  public class HashMapEx2 {
    public static void main(String[] args) {
      Map<Student, Integer> map = new HashMap<>();

      map.put(new Student(1, "홍길동"), 95);
      map.put(new Student(1, "홍길동"), 97);

      System.out.println("총 Entry 수 : " + map.size());
      System.out.println(map.get(new Student(1, "홍길동")));
    }
  }
  ```

  ```java
  총 Entry 수 : 1
  97

  Process finished with exit code 0
  ```


### Hashtable

```java
Map<K, V> map = new Hashtable<>();
```

* 특징

  * 키 객체는 hashCode()와 equals()를 재정의해서 동등객체가 될 조건을 정해야 한다.
    - 동등객체가 되려면?
      - 두 객체의 hashCode()의 리턴값이 같을 경우, 다시 두 객체의 equals()의 리턴값을 비교하여 같을 경우 동등 객체로 판단한다.
      - hashCode() 리턴값이 다르거나, hashCode() 리턴값은 같지만 equals()의 리턴값이 다른 경우 다른 객체로 판단한다.
  * Hashtable은 스레드 동기화(synchronization)가 되어 있기 때문에 복소의 스레드가 동시에 Hashtable에 접근해서 객체를 추가, 삭제하더라고 스레드레 안전(thread safe)하다.
    * ArrayList와 Vector의 관계와 같다.
    * 멀티스레드 환경에서는 Hashtable을 사용하는 것이 안전하며, 싱글스레드 환경에서는 동기화처리가 되지 않은 HashMap을 사용하는 것이 성능에 더 좋다.

* 예제

  * 기본 사용법은 HashMap이랑 같다. Thread Safe여부만 다르다.

  ```Java
  package map.hashtable;

  import java.util.Hashtable;
  import java.util.Map;
  import java.util.Scanner;

  public class HashtableEx {
    public static void main(String[] args) {
      Map<String, String> map = new Hashtable<>();

      map.put("spring", "12");
      map.put("summer", "123");
      map.put("fall", "1234");
      map.put("winter","12345");

      Scanner scanner = new Scanner(System.in);
      while(true) {
        System.out.println("아이디와 비밀번호를 입력해주세요.");
        System.out.print("아이디 :");
        String id = scanner.nextLine();
        System.out.print("비밀번호 :");
        String pw = scanner.nextLine();
        System.out.println();
        if(map.containsKey(id)) {
          if(map.get(id).equals(pw)) {
            System.out.println("로그인 되었습니다.");
          } else {
            System.out.println("비밀번호가 맞지 않습니다.");
          }
        } else {
          System.out.println("아이디가 존재하지 않습니다.");
        }
      }
    }
  }
  ```

  ```java
  아이디와 비밀번호를 입력해주세요.
  아이디 :spring
  비밀번호 :12

  로그인 되었습니다.
  아이디와 비밀번호를 입력해주세요.
  아이디 :spring
  비밀번호 :123

  비밀번호가 맞지 않습니다.
  ```


### Properties

* 특징

  * 키와 값을 String 타입으로 제한한 Map 컬렉션이다
  * Properties는 프로퍼티(~.properties) 파일을 읽어 들일 때 주로 사용한다.

* 프로퍼티 파일

  * 옵션 정보, 데이터베이스 연결정보, 국제화(다국어) 정보를 기록한 텍스트 파일로 활용
  * 애플리케이션에서 주로 변경이 잦은 문자열을 저장해서 유지 보수를 편리하게 만들어줌

  ```properties
  driver=oracle.jdbc.OracleDriver
  url=jdbc:oravle:thin@localhost:1521:orcl
  username=scott
  password=tiger
  ```

  * 키와 값이 =기호로 연결되어 있는 텍스트파일로 ISO 8859-1 문자셋으로 저장
  * 한글은 유니코드로 변환되어 저장

* Properties 객체 생성

  * 파일 시스템 경로 이용

  ```java
  Properties properties = new Properties();
  properties.load(new FileReader("C:/~/database.properties"));
  ```

  * ClassPath를 이용

  ```java
  String path = 클래스.class.getResource("database.properties").getPath();
  path = URLDecoder.decode(path, "utf-8");	//경로에 한글이 있을 경우 한글 복원
  Properties properties = new Properties();
  properties.load(new FileReader(path));
  ```

  ```java
  //패키지가 다를경우
  String path = A.class.getResource("config/database.properties").getPath();
  ```

* 값 읽기

```java
String value = properties.getProperty("key");
```

* 예제

```java
package map.properties;

import java.io.FileReader;
import java.net.URLDecoder;
import java.util.Properties;

public class PropertiesEx {
  public static void main(String[] args) throws Exception {
    Properties properties = new Properties();
    String path = PropertiesEx.class.getResource("database.properties").getPath();
    path = URLDecoder.decode(path, "utf-8");
    properties.load(new FileReader(path));

    String driver = properties.getProperty("driver");
    String url = properties.getProperty("url");

    System.out.println(driver);
    System.out.println(url);
  }
}
```

```properties
driver=oracle.jdbc.OracleDriver
url=jdbc:oravle:thin@localhost:1521:orcl
username=scott
password=tiger
```

```java
oracle.jdbc.OracleDriver
jdbc:oravle:thin@localhost:1521:orcl

Process finished with exit code 0
```

  

## 5. 검색 기능을 강화시킨 컬렉션

### 검색기능을 강화시킨 컬렉션

* TreeSet, TreeMap
  * 이진트리(binary tree)를 사용하기 때문에 검색 속도 향상

### 이진 트리 구조

* 부모 노드와 자식 노드로 구성
  * 왼쪽 자식 노드 : 부모 보다 작은 값
  * 오른쪽 자식 노드 : 부모 보다 큰 값
* 정렬이 쉬움
  * 오름 차순 : 왼쪽노드 -> 부모노드 -> 오른쪽노드
  * 내림 차순 : 오른쪽노드 -> 부모노드 -> 왼쪽노드
* 범위 검색이 쉽다
  * 노드를 기준으로 왼쪽 자식트리(작은 값들)와 오른쪽 자식트리(큰 값들)의 속성이 결정되어있음

### TreeSet

```java
TreeSet<E> treeSet = new TreeSet<>();
```

TreeSet은 Set 인터페이스의 구현 클래스이므로 Set인터페이스의 타입의 변수에 대입해도 되지만, TreeSet 타입의 변수에 대입한 이유는 TreeSet이 가지고 있는 검색 기능 메소드를 사용하기 위함이다.

* 특징

  * 이진 트리(Binary Tree)를 기반으로 한 Set 컬렉션
  * 왼쪽과 오른쪽 자식노드를 참조하기 위한 두개의 변수로 구성

* 주요 메소드

  * 특정 객체를 찾는 메소드

    * first(), last(), lower(), higher(), ...

    ```java
    package advanced_search_collection.treeset;

    import java.util.Iterator;
    import java.util.TreeSet;

    public class TreeSetEx01 {
      public static void main(String[] args) {
        TreeSet<Integer> scores = new TreeSet<>();

        scores.add(87);
        scores.add(98);
        scores.add(75);
        scores.add(95);
        scores.add(80);

        System.out.println("최소값(first): " + scores.first());
        System.out.println("최대값(last): " + scores.last());
        System.out.println("95 아래에 있는 값(미만, lower): " + scores.lower(95));
        System.out.println("95 위에 있는 값(초과, higher): " + scores.higher(95));
        System.out.println("95이거나 바로 아래 있는 값(이하, floor): " + scores.floor(95));
        System.out.println("95이거나 바로 위에 있는 값(이상, ceiling): " + scores.ceiling(95));
        System.out.println();

        System.out.println("Iterator");
        Iterator<Integer> iterator = scores.iterator();
        while(iterator.hasNext()) {
          System.out.println(iterator.next());
        }
        System.out.println();

        System.out.println("최소값(pollFirst)/최대값(pollLast) 추출 후 set에서 제거");
        while (!scores.isEmpty()) {
          int score = scores.pollFirst();
    //      int score = scores.pollLast();
          System.out.print("빼낸 값: " + score);
          System.out.println(" / 남은 객체 수: " + scores.size());
        }
        System.out.println();
      }
    }
    ```

    ```java
    최소값(first): 75
    최대값(last): 98
    95 아래에 있는 값(미만, lower): 87
    95 위에 있는 값(초과, higher): 98
    95이거나 바로 아래 있는 값(이하, floor): 95
    95이거나 바로 위에 있는 값(이상, ceiling): 95

    Iterator
    75
    80
    87
    95
    98

    최소값(pollFirst)/최대값(pollLast) 추출 후 set에서 제거
    빼낸 값: 75 / 남은 객체 수: 4
    빼낸 값: 80 / 남은 객체 수: 3
    빼낸 값: 87 / 남은 객체 수: 2
    빼낸 값: 95 / 남은 객체 수: 1
    빼낸 값: 98 / 남은 객체 수: 0
    ```


    Process finished with exit code 0
    ​```


  * 정렬 메소드

    * descendingIterator(), descendingSet()

    ```java
    package advanced_search_collection.treeset;

    import java.util.NavigableSet;
    import java.util.TreeSet;

    public class TreeSetEx02 {
      public static void main(String[] args) {
        TreeSet<Integer> scores = new TreeSet<>();

        scores.add(87);
        scores.add(98);
        scores.add(75);
        scores.add(95);
        scores.add(80);
        System.out.println(scores);

        NavigableSet<Integer> descendingSet = scores.descendingSet();
        System.out.println("descendingSet");
        for(int score : descendingSet) {
          System.out.print(score + " ");
        }

        System.out.println();
        System.out.println("ascendingSet");
        NavigableSet<Integer> ascendSet = descendingSet.descendingSet();
        for(int score : ascendSet) {
          System.out.print(score + " ");
        }
      }
    }
    ```

    ```java
    [75, 80, 87, 95, 98]
    descendingSet
    98 95 87 80 75 
    ascendingSet
    75 80 87 95 98 
    Process finished with exit code 0
    ```

  * 범위 검색 메소드

    * headSet(), tailSet, subSet()

    ```java
    package advanced_search_collection.treeset;

    import java.util.NavigableSet;
    import java.util.TreeSet;

    public class TreeSetEx03 {
      public static void main(String[] args) {
        TreeSet<String> treeSet = new TreeSet<>();

        treeSet.add("apple");
        treeSet.add("forever");
        treeSet.add("description");
        treeSet.add("ever");
        treeSet.add("zoo");
        treeSet.add("base");
        treeSet.add("guess");
        treeSet.add("cherry");
        treeSet.add("f");
        treeSet.add("c");

        System.out.println("[c~f사이의 단어 검색]");
        NavigableSet<String> rangeSet = treeSet.subSet("c", true, "f", true);
        System.out.println(rangeSet);
      }
    }
    ```

    ```java
    [c~f사이의 단어 검색]
    [c, cherry, description, ever, f]

    Process finished with exit code 0
    ```



### TreeMap

```java
TreeMap<K, V> treeMap = new TreeMap<K, V>();
```

검색속도를 향상 시키기 위해서 제공되는 컬렉션, 제공되는 객체를 MapEntry형태로 제공하는 컬렉션

- 특징

  - 이진 트리(Binary Tree)를 기반으로한 Map 컬렉션
  - 키와 값이 저장된 Map.Entry를 저장
  - 왼쪽과 오른쪽 자식노드를 참조하기 위한 두개의 변수로 구성

- 주요 메소드

  - 단일 노드 객체를 찾는 메소드

    - firstEntry(), lastEntry(), lowerEntry(), lowerEntry(), higherEntry(), ...

    ```java
    package advanced_search_collection.treemap;

    import java.util.Map;
    import java.util.TreeMap;

    public class TreeMapEx01 {
      public static void main(String[] args) {
        TreeMap<Integer, String> scores = new TreeMap<>();
        scores.put(87, "홍길동");
        scores.put(98, "이동수");
        scores.put(75, "박길순");
        scores.put(95, "신용권");
        scores.put(80, "김자바");

        Map.Entry<Integer, String> entry = scores.firstEntry();
        System.out.println("가장 작은 키값을 가진 객체: " + entry.getKey() + ", " + entry.getValue());

        entry = scores.lastEntry();
        System.out.println("가장 큰 키값을 가진 객체: " + entry.getKey() + ", " + entry.getValue());

        entry = scores.lowerEntry(95);
        System.out.println("95점 미만의 키값을 가진 객체: " + entry.getKey() + ", " + entry.getValue());

        entry = scores.higherEntry(95);
        System.out.println("95점 초과의 키값을 가진 객체: " + entry.getKey() + ", " + entry.getValue());

        entry = scores.floorEntry(95);
        System.out.println("95점 이상의 키값을 가진 객체: " + entry.getKey() + ", " + entry.getValue());

        entry = scores.ceilingEntry(95);
        System.out.println("95점 이하의 키값을 가진 객체: " + entry.getKey() + ", " + entry.getValue());

        System.out.println("키의 값이 최소값(pollFirst)/최대값(pollLast)인 객체 추출 후 Map에서 제거");
        while (!scores.isEmpty()) {
          //entry = scores.pollFirstEntry();
          entry = scores.pollLastEntry();
          System.out.println(entry.getKey() + ", " + entry.getValue() + "(남은 객체 수: " + scores.size() + ")");
        }
      }
    }
    ```

    ```java
    가장 작은 키값을 가진 객체: 75, 박길순
    가장 큰 키값을 가진 객체: 98, 이동수
    95점 미만의 키값을 가진 객체: 87, 홍길동
    95점 초과의 키값을 가진 객체: 98, 이동수
    95점 이상의 키값을 가진 객체: 95, 신용권
    95점 이하의 키값을 가진 객체: 95, 신용권

    키의 값이 최소값(pollFirst)/최대값(pollLast)인 객체 추출 후 Map에서 제거
    98, 이동수(남은 객체 수: 4)
    95, 신용권(남은 객체 수: 3)
    87, 홍길동(남은 객체 수: 2)
    80, 김자바(남은 객체 수: 1)
    75, 박길순(남은 객체 수: 0)

    Process finished with exit code 0
    ```


  - 정렬 메소드

    - descendingKetSet(), descendingMap()

    ```java
    package advanced_search_collection.treemap;

    import java.util.Map;
    import java.util.NavigableMap;
    import java.util.Set;
    import java.util.TreeMap;

    public class TreeMapEx02 {
      public static void main(String[] args) {
        TreeMap<Integer, String> scores = new TreeMap<>();
        scores.put(87, "홍길동");
        scores.put(98, "이동수");
        scores.put(75, "박길순");
        scores.put(95, "신용권");
        scores.put(80, "김자바");
        System.out.println(scores);

        NavigableMap<Integer, String> descendingMap = scores.descendingMap();
        Set<Map.Entry<Integer, String>> descendingEntrySet = descendingMap.entrySet();
        for(Map.Entry<Integer, String> entry : descendingEntrySet) {
          System.out.print(entry.getKey() + "-" + entry.getValue() + " ");
        }
        System.out.println();

        NavigableMap<Integer, String> ascendingMap = descendingMap.descendingMap();
        Set<Map.Entry<Integer, String>> ascendingEntrySet = ascendingMap.entrySet();
        for(Map.Entry<Integer, String> entry : ascendingEntrySet) {
          System.out.print(entry.getKey() + "-" + entry.getValue() + " ");
        }
        System.out.println();
      }
    }
    ```

    ```java
    {75=박길순, 80=김자바, 87=홍길동, 95=신용권, 98=이동수}
    98-이동수 95-신용권 87-홍길동 80-김자바 75-박길순 
    75-박길순 80-김자바 87-홍길동 95-신용권 98-이동수 

    Process finished with exit code 0
    ```

  - 범위 검색 메소드

    - headMap(), tailMap(), subMap()

    ```java
    package advanced_search_collection.treemap;

    import java.util.Map;
    import java.util.NavigableMap;
    import java.util.TreeMap;

    public class TreeMapEx03 {
      public static void main(String[] args) {
        TreeMap<String, Integer> treeMap = new TreeMap<>();
        treeMap.put("apple", 10);
        treeMap.put("forever", 60);
        treeMap.put("description", 40);
        treeMap.put("ever", 50);
        treeMap.put("zoo", 10);
        treeMap.put("base", 20);
        treeMap.put("guess", 70);
        treeMap.put("cherry", 30);
        treeMap.put("f", 5);
        treeMap.put("c", 80);

        System.out.println("[c~f 사이의 단어 검색]");
        NavigableMap<String, Integer> rangeMap = treeMap.subMap("c", true, "f", true);
        for (Map.Entry<String, Integer> entry : rangeMap.entrySet()) {
          System.out.print(entry.getKey() + "-" + entry.getValue() + " ");
        }
      }
    }
    ```

    ```java
    [c~f 사이의 단어 검색]
    c-80 cherry-30 description-40 ever-50 f-5 
    Process finished with exit code 0
    ```



### Comparable과 Comparator

* TreeSet과 TreeMap의 자동 정렬

  * TreeSet의 객체와 TreeMap의 키는 저장과 동시에 자동 오름차순으로 정렬
  * 숫자(Integer, Double)타입일 경우에는 값으로 정렬
  * 문자열(String)타입일 경우에는 값으로 정렬
  * TreeSet과 TreeMap은 정렬을 위해 java.lang.Comparable을 구현한 객체를 요구
    * Integer, Double, String은 모두 Comparable 인터페이스를 구현한 클래스이기 때문에 TreeSet과 TreeMap에 저장할 때, 자동으로 오름차순으로 정렬될 수 있다.
    * 사용자가 정의하는 객체를 TreeSet, TreeMap에 저장하려고 할 때, Comparable을 구현하고 있지 않을 경우에는 저장하는 순간 ClassCastException이 발생

* 사용자 정의 객체를 정렬하고 싶을 경우

  * **방법1 : 사용자 정의 클래스가 Comparable을 구현**

  | 리턴타입 | 메소드            | 설명                                       |
  | ---- | -------------- | ---------------------------------------- |
  | int  | compareTo(T o) | 주어진 객체과 같으면 0을 리턴, 주어진 객체보자 적으면 음수를 리턴, 주어진 객체보다 크면 양수를 리턴 |

  ```java
  package comparable;

  public class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
      this.name = name;
      this.age = age;
    }

    @Override
    public int compareTo(Person o) {
      if (age < o.getAge()) return -1;
      else if (age == o.age) return 0;
      else return 1;
    }

    public String getName() {
      return name;
    }

    public int getAge() {
      return age;
    }

    @Override
    public String toString() {
      return age + "-" + name;
    }
  }
  ```

  ```java
  package comparable;

  import java.util.Iterator;
  import java.util.TreeSet;

  public class ComparableEx {
    public static void main(String[] args) {
      TreeSet<Person> treeSet = new TreeSet<>();
      treeSet.add(new Person("김남준", 25));
      treeSet.add(new Person("신용권", 40));
      treeSet.add(new Person("김자바", 30));
      System.out.println(treeSet);

      Iterator<Person> iterator = treeSet.iterator();
      while(iterator.hasNext()) {
        Person person = iterator.next();
        System.out.println(person);
      }
    }
  }
  ```

  ```java
  [25-김남준, 30-김자바, 40-신용권]
  25-김남준
  30-김자바
  40-신용권

  Process finished with exit code 0
  ```

  * **방법2 : TreeSet, TreeMap 생성시 Comparator 구현 객체 제공**

  | 리턴타입 | 메소드                 | 설명                                       |
  | ---- | ------------------- | ---------------------------------------- |
  | int  | compare(T o1, T o2) | o1과 o2가 동등하다면 0을 리턴, o1이 o2보다 앞에 오게 하려면 음수를 리턴, o1과 o2보다 뒤에 오게 하려면 양수를 리턴 |

  ```java
  package comparator;

  public class Fruit {
    private String name;
    private int price;

    public Fruit(String name, int price) {
      this.name = name;
      this.price = price;
    }

    public String getName() {
      return name;
    }

    public int getPrice() {
      return price;
    }

    @Override
    public String toString() {
      return price + "-" + name;
    }
  }
  ```

  ```java
  package comparator;

  import java.util.Comparator;

  public class DescendingComparator implements Comparator<Fruit> {

    @Override
    public int compare(Fruit o1, Fruit o2) {
      if (o1.getPrice() < o2.getPrice()) return -1;
      else if (o1.getPrice() == o2.getPrice()) return 0;
      else return 1;
    }
  }
  ```

  ```java
  package comparator;

  import java.util.Iterator;
  import java.util.TreeSet;

  public class ComparatorEx {
    public static void main(String[] args) {
      TreeSet<Fruit> treeSet = new TreeSet<>(new DescendingComparator());

      treeSet.add(new Fruit("포도", 3000));
      treeSet.add(new Fruit("수박", 10000));
      treeSet.add(new Fruit("딸기", 6000));
      System.out.println(treeSet);

      Iterator<Fruit> iterator = treeSet.iterator();
      while(iterator.hasNext()) {
        Fruit fruit = iterator.next();
        System.out.println(fruit);
      }
    }
  }
  ```

  ```java
  [3000-포도, 6000-딸기, 10000-수박]
  3000-포도
  6000-딸기
  10000-수박

  Process finished with exit code 0
  ```


## 6. LIFO와 FIFO 컬렉션

Last In First Out, First In First Out

### Stack 클래스

```Java
Stack<E> stack = new Stack<>();
```

* 특징
  * 후입선출(Last In First Out) 구조
  * 응용 예: JVM 스택 메모리

* 주요 메소드

  | 리턴타입 | 메소드          | 설명                                  |
  | ---- | ------------ | ----------------------------------- |
  | E    | push(E item) | 주어진 객체를 스택에 넣는다                     |
  | E    | peek()       | 스택의 맨위 객체를 가져온다. 객체를 스택에서 제거하지 않는다. |
  | E    | pop()        | 스택의 맨위 객체를 가져온다. 객체를 스택에서 제거한다.     |

* 예제 코드

```java
package stack;

public class Coin {
  private int value;

  public Coin(int value) {
    this.value = value;
  }

  public int getValue(){
    return value;
  }
}
```

```java
package stack;

import java.util.Stack;

public class StackEx {
  public static void main(String[] args) {
    Stack<Coin> coinBox = new Stack<>();

    coinBox.push(new Coin(100));
    coinBox.push(new Coin(500));
    coinBox.push(new Coin(50));

    while(!coinBox.isEmpty()) {
      System.out.println("꺼낸 동전 : " + coinBox.pop().getValue() + "원");
    }
  }
}
```

```java
꺼낸 동전 : 50원
꺼낸 동전 : 500원
꺼낸 동전 : 100원

Process finished with exit code 0
```

### Queue 클래스

```java
Queue queue = new LinkedList();
```

* 특징

  * 선입선출(First In First Out) tnwh
  * 응용 예: 작업큐, 메세지큐, ...
  * 구현 클래스 : LinkedList

* 주요 메소드

  | 리턴타입    | 메소드        | 설명                             |
  | ------- | ---------- | ------------------------------ |
  | boolean | offer(E e) | 주어진 객체를 넣는다.                   |
  | E       | peek()     | 객체 하나를 가져온다. 객체를 큐에서 제거하지 않는다. |
  | E       | poll()     | 객체 하나를 가져온다. 객체를 큐에서 제거한다.     |

* 예제 코드

```java
package queue;

public class Message {
  private String command;
  private String to;

  public Message(String command, String to) {
    this.command = command;
    this.to = to;
  }

  public String getCommand() {
    return command;
  }

  public String getTo() {
    return to;
  }
}
```

```java
package queue;

import java.util.LinkedList;
import java.util.Queue;

public class QueueEx {
  public static void main(String[] args) {
    Queue<Message> messageQueue = new LinkedList<>();

    messageQueue.offer(new Message("send Email", "홍길동"));
    messageQueue.offer(new Message("send SMS", "신용권"));
    messageQueue.offer(new Message("send Kakaotalk", "김남준"));

    while(!messageQueue.isEmpty()) {
      Message message = messageQueue.poll();
      switch(message.getCommand()) {
        case "send Email":
          System.out.println(message.getTo() + "님에게 메일을 보냅니다.");
          break;
        case "send SMS":
          System.out.println(message.getTo() + "님에게 SMS를 보냅니다.");
          break;
        case "send Kakaotalk":
          System.out.println(message.getTo() + "님에게 KakaoTalk 보냅니다.");
          break;
      }
    }
  }
}
```

```java
홍길동님에게 메일을 보냅니다.
신용권님에게 SMS를 보냅니다.
김남준님에게 KakaoTalk 보냅니다.

Process finished with exit code 0
```

  

## 7. 동기화된(synchronized) 컬렉션

### 비동기화된 컬렉션을 동기화된 컬렉션으로 래핑

비동기화된 컬렉션을 멀티스레드 환경에서 사용해야 할 경우 동기화된 컬렉션으로 래핑하여 사용하는 것이 안전하다.

#### List 컬렉션

```java
List<T> list = Collections.synchronizedList(new ArrayList<T>());
```

* 위와 같이 비동기화된 컬렉션인 ArrayList를 Collections클래스의 정적메소드를 통해서 래핑시켜주면 Thread Safe한 컬렉션을 만들 수 있다.

#### Set 컬렉션

```java
Set<E> set = Collections.synchronizedSet(new HashSet<E>);
```

* 마찬가지로 비동기화된 컬렉션인 HashSet를 Collections클래스의 정적메소드를 통해서 래핑시켜주면 Thread Safe하게 만들 수 있다.

#### Map 컬렉션

```java
Map<K, V> map = Collections.synchronizedMap(new HashMap<K, V>));
```

* 마찬가지로 비동기화된 컬렉션인 HashMap를 Collections클래스의 정적메소드를 통해서 래핑시켜주면 Thread Safe하게 만들 수 있다.