# 13장. 제네릭

>[소스코드 repo](https://github.com/namjunemy/this_is_java)
>
>1절. 왜 제네릭을 사용해야 하는가?
>
>2절. 제네릭 타입
>
>3절. 멀티 타입 파라미터
>
>4절. 제네릭 메소드
>
>5절. 제한된 타입 파라미터
>
>6절. 와일드카드 타입
>
>7절. 제네릭 타입의 상속과 구현



## 1절. 왜 제네릭을 사용해야 하는가?
#### 제네릭(Generic) 이란

* 타입을 파라미터화해서 컴파일시 구체적인 타입이 결정되도록 하는 것
  * 자바 5부터 추가된 기능이다
  * 컬렉션, 람다식(함수적 인터페이스), 스트링, NIO에서 널리 사용된다
  * 제네릭을 모르면 도큐먼트를 해석할 수 없다.

```
Class Arraylist<E>

default BiConsumer<T,U> andThen(BiConsumer<? super T,? super I> after)
```
위의 코드에서 `E, T, U, ?`가 무엇을 뜻하느냐가 이번 장을 학습하면 이해가 된다.  

  

#### 제네릭을 사용하므로서 얻는 이점

- 컴파일시 강한 타입체크를 할 수 있다.	 
   - 실행시 타입 에러가 나는 것보다는 컴파일시에 미리 타입을 강하게 체크해서 에러를 사전에 방지한다.

- 타입변환을 제거할 수 있다.

     - 타입 변환이 많이 생길수록 전체 애플리케이션 성능이 떨어지게 된다.

     ```java
     List list = new ArrayList();
     list.add("hello");
     String str = (String) list.get(0);
     ```

     * 위의 코드에서는 강제타입 변환이 두번 일어난다.	
     * "hello"라는 String 객체를 Object 타입으로 저장할 때 한번, list.get(0)의 반환값인 Object 타입의 객체를 String 타입으로 저장할 때 한번. 제네릭을 적용하면 다음과 같이 코드가 변화한다.	

     ```java
     List<String> list = new ArrayList<String>();
     list.add("hello");
     String str = list.get(0);
     ```

     * List 객체를 생성할 때, String 객체를 저장하겠다고 선언하면 불필요한 타입변환이 사라지게 된다.	

       


## 2절. 제네릭 타입
#### 제네릭 타입이란

- 타입을 파라미터로 가지는 클래스와 인터페이스를 말한다
   - 선언시 클래스 또는 인터페이스 이름 뒤에 "< >"부호가 붙는다.
  - "< >" 사이에는 타입 파라미터가 위치한다.

 ```java
 public class 클래스명<T> { ... }	
 
 public interface 인터페이스명<T> { ... }
 ```
- 타입 파라미터
   - 일반적으로 대문자 알파벳 한 문자로 표현한다.
   - 개발 코드에서는 타입 파라미터 자리에 구체적인 타입을 지정해야 한다.


- 제네릭 타입을 사용할 경우의 효과
   * 비 제네릭
      * 1절의 코드에서 처럼, 모든 객체의 최상위 객체인 Object 타입을 사용하므로서 빈번한 타입 변환 발생 -> 성능 저하
    * 제네릭
       * 클래스를 선언할 때 타입 파라미터를 기술해 준다.
       * 따라서, 컴파일시 타입 파라미터가 구체적인 클래스로 변경된다. 불필요한 타입변환이 이루어지지 않는다.
         * 아래와 같이 코드가 변경되면,
    ```java
   public class Box<T> {
     private T t;

     public T get() { return t; }

     public void set(T t) { this.t = t }
   }
    ```
    String 객체일 경우,

    ```java	 
   Box<String> box = new Box<String>();
    ```
    ```java
   public class Box<String> {
     private String t;

     public String get() { return t; }

     public void set(String t) { this.t = t }
   }
    ```
    Integer 객체의 경우,

    ```java	 
   Box<Integer> box = new Box<Integer>();
    ```
    ```java	 
   public class Box<Integer> {
     private Integer t;

     public Integer get() { return t; }

     public void set(Integer t) { this.t = t }
   }
    ```
   * 위와 같이, 클래스를 설계할 때 타입파라미터로 설계를 해 두고, 실제로 사용할 때 구체적인 클래스를 지정해 줌으로써 컴파일러가 클래스를 재구성해준다.
   * 따라서, 전혀 타입변환이 생기지 않는다.
     * 한가지 더 중요한 사실은 실제 사용시 구체적인 클래스를 지정해 줌으로써 컴파일러가 클래스를 재구성 했을 때, 강한타입 체크를 하게 되므로 사전에 컴파일 에러를 방지한다.	
   * 예를 들어, 클래스 사용시 제네릭타입에 String 타입을 설정 했다면 Integer 타입이 들어올 경우 에러가 컴파일 에러가 발생한다. 이처럼 사전에 에러를 방지 할 수 있다.
     * 결과적으로 제네릭을 사용하는것이 애플리케이션 성능을 좋게 만들 수 있다.



## 3절. 멀티타입 파라미터
- 두 개 이상의 타입 파라미터를 사용해서 선언할 수 있다.	

  ```java
  class<K,V,...> { ... }
  interface<K,V,...> { ... }
  ```

  ```java
  Product<TV, String> product = new Product<Tv,String>();
  ```
- 중복된 타입 파라미터를 생략한 다이아몬드(<>) 연산자
   * 자바7부터 지원

 ```java
Product<Tv,String> product = new Product<>();
 ```


## 4절. 제네릭 메소드
- 매개변수 타입과 리턴 타입으로 타입 파라미터를 갖는 메소드를 말한다.

- 제네릭 메소드 선언 방법
   - 리턴 타입 앞에 "< >"기호를 추가하고 타입 파라미터를 기술한다.
   - 타입 파라미터를 리턴타입(Box\<T>)과 매개변수(T)에 사용한다.

 ```java
 public <타입파라미터,...> 리턴타입 메소드명(매개변수,...) { ... }
 
 public <T> Box<T> boxing(T t) { ... }
 ```
- 제네릭 메소드를 호출하는 두가지 방법

  ```java
  1. 리턴타입 변수 = <구체적 타입> 메소드명(매개값);	//명시적으로 구체적 타입 지정
  2. 리턴타입 변수 = 메소드명(매개값);	//매개값을 보고 구체적 타입을 추정
  ```

  ```java
  1. Box<Integer> box = <Integer>boxing(100);	//타입 파라미터를 명시적으로 Integer로 지정
  2. Box<Integer> box = boxing(100); 	//타입 파라미터를 Integer로 추정
  ```
  * 일반적으로 매개값을 넣어줌으로 컴파일러가 유추하게 만들어주는 두번째 방법을 사용한다.	


#### 실습예제

```java
public class Pair<K, V> {
  private K key;
  private V value;

  public Pair(K key, V value) {
    this.key = key;
    this.value = value;
  }
  public void setKey(K key) {
    this.key = key;
  }

  public void setValue(V value) {
    this.value = value;
  }

  public K getKey() {
    return key;
  }

  public V getValue() {
    return value;
  }
}
```

```java
public class Util {
  public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {
    boolean keyCompare, valueCompare;

    keyCompare = p1.getKey().equals(p2.getKey());
    valueCompare = p1.getValue().equals(p2.getValue());

    return keyCompare && valueCompare;
  }
}
```

```java
public class CompareMethodEx {
  public static void main(String[] args) {
    Pair<String, Integer> p1 = new Pair<>("김남준", 3);
    Pair<String, Integer> p2 = new Pair<>("김남준", 3);

    boolean result = Util.compare(p1, p2);
    System.out.println(result);

    Pair<String, String> p3 = new Pair<>("김남준", "김남준");
    Pair<String, String> p4 = new Pair<>("김남준", "준남김");

    result = Util.compare(p3, p4);
    System.out.println(result);
  }
}
```
```java
true
false
```

## 5. 제한된 타입 파라미터
#### 타입 파라미터에 지정되는 구체적인 타입을 제한할 필요가 있을 경우

 - 상속 및 구현 관계를 이용해서 타입을 제한

  ```java
   public <T extends 상위타입> 리턴타입 메소드(매개변수, ...) { ... } 
  ```
  * 상위 타입은 클래스 뿐만 아니라 인터페이스도 가능하다. **인터페이스라고 해서 extends 대신 implements를 사용하지 않는다.**

- 타입 파라미터를 대체할 구체적인 타입

  - 상위타입이거나 하위 또는 구현 클래스만 지정할 수 있다.

- 주의할 점
   - 메소드의 중괄호 {} 안에서 타입 파라미터 변수로 사용 가능한 것은 상위 타입의 멤버(필드, 메소드)로 제한된다.
  - 하위 타입에만 있는 필드와 메소드는 사용할 수 없다.
     -> 상위타입으로 타입 파라미터를 제한시킨 상태에서 하위 타입의 멤버를 사용하면, 상위타입이 들어올 경우 에러가 발생한다.

   ```java
   public <T extends Number> int compare(T t1, T t2) {
       double v1 = t1.doubleValue();
       double v2 = t2.doubleValue();
       
       return Double.compare(v1, v2);
   }
   ```

#### 실습예제

```java
public class Util {
  public static <T extends Number> int compare(T t1, T t2) {
    double v1 = t1.doubleValue();
    double v2 = t2.doubleValue();

    return Double.compare(v1, v2);
  }
}
```

```java
public class BoundedTypeParameterEx {
  public static void main(String[] args) {
    int result1 = Util.compare(8, 4);
    System.out.println(result1);

    result1 = Util.compare(3, 8);
    System.out.println(result1);
  }
}
```

## 6. 와일드카드 타입
**제네릭 타입을 매개변수나 리턴타입으로 사용할 때 타입 파라미터를 제한할 목적**

- `<T extends 상위 또는 인터페이스>`는 제네릭 타입과 제네릭 메소드를 선언할 때 제한을 한다.

  ```java
  public static void registerCourse(Course<?> course) {
  public static void registerCourseStudent(Course<? extends Student> course) {
  public static void registerCourseWorker(Course<? super Worker> course) {
  ```

#### 와일드카드 타입의 세가지 형태

 * `제네릭타입<?>` : Unbounded Wildcards (제한없음)	
   * 타입 파라미터를 대치하는 구체적인 타입으로 모든 클래스나 인터페이스 타입이 올 수 있다.

 * `제네릭타입<? extends 상위타입>` : Upper Bounded Wildcards (상위 클래스 제한)
  * 타입 파라미터를 대치하는 구체적인 타입으로 상위 타입이나, 그 상위 타입의 하위 타입만 올 수 있다. 따라서, **상위 클래스 제한** 이라고 한다.	
  * 즉, 상위 타입이 해당 자리에 들어갈 수 있는 가장 상위 타입이다.

 * `제네릭타입<? super 하위타입>` : Lower Bounded Wildcards (하위 클래스 제한)
  * 타입 파라미터를 대치하는 구체적인 타입으로 하위 타입이나, 그 하위 타입의 상위 타입이 올 수 있다. 따라서, **하위 클래스 제한** 이라고 한다.	
  * 즉, 하위 타입이 해당 자리에 들어갈 수 있는 가장 하위 타입이다.


#### 실습예제

```java
public class Course<T> {
  private String name;
  private T[] students;

  public Course(String name, int capacity) {
    this.name = name;
    students = (T[]) (new Object[capacity]);
  }

  public String getName() {
    return name;
  }

  public T[] getStudents() {
    return students;
  }

  public void add(T t) {
    for (int i = 0; i < students.length; i++) {
      if (students[i] == null) {
        students[i] = t;
        break;
      }
    }
  }
}
```

```java
public class Person {
  private String name;

  public Person(String name) {
    this.name = name;
  }

  public String getName() {
    return name;
  }

  @Override
  public String toString() {
    return name;
  }
}
```

```java
public class Student extends Person {
  public Student(String name) {
    super(name);
  }
}
```

```java
public class Worker extends Person {
  public Worker(String name) {
    super(name);
  }
}
```

```java
public class HighStudent extends Student {
  public HighStudent(String name) {
    super(name);
  }
}
```

```java
public class WildCardEx {
  public static void registerCourse(Course<?> course) {
    System.out.println(course.getName() + " 수강생 : " + Arrays.toString(course.getStudents()));
  }

  public static void registerCourseStudent(Course<? extends Student> course) {
    System.out.println(course.getName() + " 수강생 : " + Arrays.toString(course.getStudents()));
  }

  public static void registerCourseWorker(Course<? super Worker> course) {
    System.out.println(course.getName() + " 수강생 : " + Arrays.toString(course.getStudents()));
  }

  public static void main(String[] args) {
    Course<Person> personCourse = new Course<>("일반인 과정", 5);
    personCourse.add(new Person("일반인"));
    personCourse.add(new Person("직장인"));
    personCourse.add(new Person("학생"));
    personCourse.add(new Person("고등학생"));

    Course<Worker> workerCourse = new Course<>("직장인 과정", 5);
    workerCourse.add(new Worker("직장인"));

    Course<Student> studentCourse = new Course<>("학생 과정", 5);
    studentCourse.add(new Student("학생"));
    studentCourse.add(new HighStudent("고등학생"));

    Course<HighStudent> highStudentCourse = new Course<>("고등학생 과정", 5);
    highStudentCourse.add(new HighStudent("고등학생"));

    registerCourse(personCourse);
    registerCourse(workerCourse);
    registerCourse(studentCourse);
    registerCourse(highStudentCourse);

    System.out.println();

    registerCourseStudent(studentCourse);
    registerCourseStudent(highStudentCourse);

    System.out.println();

    registerCourseWorker(workerCourse);
    registerCourseWorker(personCourse);

    System.out.println();
  }
}
```

## 7절. 제네릭 타입의 상속과 구현
#### 제네릭 타입을 부모 클래스로 사용할 경우

- 타입 파라미터는 자식 클래스에도 기술해야 한다.	

  ```java
  public class ChildProduct<T,M> extends Product<T,M> { ... }
  ```

  - 추가적인 타입 파라미터를 가질 수 있다.	

  ```java
  public class ChildProduct<T,M,C> extends Product<T,M> { ... }
  ```

#### 제네릭 인터페이스를 구현할 경우

- 타입 파라미터는 구현 클래스에도 기술해야 한다.	

  ```java
  public class StorageImpl<T> implements Storage<T> { ... }
  ```