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

