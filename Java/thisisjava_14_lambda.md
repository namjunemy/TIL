## 14장. 람다식

> [소스코드 repo](https://github.com/namjunemy/this_is_java)
>
> 1절. 람다식이란
>
> 2절. 람다식 기본 문법
>
> 3절. 타겟 타입과 함수적 인터페이스
>
> 4절. 외부 로컬 변수 접근
>
> 5절. 기본적으로 제공되는 함수적 인터페이스
>
> 6절. 메소드 참조

  

## 1절. 람다식이란

#### 함수적 프로그래밍

* y = f(x) 형태의 함수로 구성된 프로그래밍 기법
  * 데이터를 매개값으로 전달하고 결과를 받는 코드들로 구성된다.
  * 객체 지향 프로그래밍 보다 효율적인 경우
    * 대용량 데이터의 처리시에 유리
      * 데이터 포장 객체를 생성 후 처리하는 것 보다, 데이터를 바로 처리하는 것이 속도에 유리
      * 멀티 코어 CPU에서 데이터를 병렬 처리하고 취합할 때 객체보다는 함수가 유리
    * 이벤트 지향 프로그래밍(이벤트가 발생하면 핸들러 함수 실행)에 적합
      * 반복적인 이벤트 처리는 핸들러 객체보다는 핸들러 함수가 적합
* 현대적 프로그래밍 기법
  * 객체 지향 프로그래밍 + 함수적 프로그래밍

  

#### 자바는 8부터 함수적 프로그래밍 지원

* 람다식(Lambda Expressions)을 언어 차원에서 제공

  * 람다 계산법에서 사용된 식을 프로그래밍 언어에 접목
  * 익명 함수(anonymous function)을 생성하기 위한 식
    * ```(타입 매개변수, ...) -> { 실행문; ...}```

* 자바에서 람다식을 수용한 이유

  * 코드가 매우 간결해진다.
  * 컬렉션 요소(대용량 데이터)를 필터링 또는 매핑해서 쉽게 집계할 수 있다.

* 자바는 람다식을 **함수적 인터페이스의 익명 구현 객체**로 취급한다.

  * ```람다식 -> 매개변수를 가진 코드 블록 -> 익명 구현 객체```
  * 어떤 인터페이스를 구현할지는 대입되는 인터페이스에 달려있다.

  ```java
  Runnable runnable = () -> { ... }; // 람다식
  ```

  ```java
  Runnable runnable = new Runnable() {
    public void run() {
        ...
    }
  };
  ```

  

## 2절. 람다식 기본 문법

* 함수적 스타일의 람다식을 작성하는 방법
  * (타입 매개변수, …) -> { 실행문; ...}
    * ex) (int a) -> { System.out.println(a); }
* 람다식의 6가지 약식 표현
  * 매개 타입은 런타임시에 대입값에 따라 자동으로 인식하기 때문에 생략 가능하다.
    * 람다식은 함수적 인터페이스의 메소드를 구현한 것이기 때문에, 아래에서 매개변수만 지정을 해도 함수적 인터페이스의 메소드의 타입을 보고 a의 타입을 결정 할 수 있다. 따라서 a의 타입을 생략 할 수 있다.
    * ```(a) -> { System.out.println(a); }```
  * 하나의 매개변수만 있을 경우에는 괄호 () 생략 가능
    * ```a -> { System.out.println(a); }```
  * 하나의 실행문만 있다면 중괄호 {} 생략 가능
    * ```a -> System.out.println(a)```
  * 매개변수가 없다면 괄호 ()를 생략할 수 없음
    * ```() -> { 실행문; ... }```
  * 리턴값이 있는 경우, return 문을 사용
    * ```(x,y) -> { return x + y; };```
  * 중괄호 {}에 return 문만 있을 경우, 중괄호를 생략 가능
    * ```(x,y) -> x + y```

  

## 3절. 타겟 타입과 함수적 인터페이스

* 타겟 타입(target type)

  * 람다식이 대입되는 인터페이스를 말한다.
    * 인터페이스 변수 = 람다식;
  * 익명 구현 객체를 만들 때 사용할 인터페이스이다.

* 함수적 인터페이스(functional interface)

  * 모든 인터페이스는 람다식의 타겟 타입이 될 수 없다.
    * 람다식은 하나의 메소드를 정의하기 떄문에
    * 하나의 추상 메소드만 선언된 인터페이스만 타겟 타입이 될 수 있다.
  * 함수적 인터페이스
    * 하나의 추상 메소드만 선언된 인터페이스를 말한다.
  * @FunctionalInterface 어노테이션
    * 하나의 추상 메소드만을 가지는지 컴파일러가 체크하도록 함
    * 두 개 이상의 추상 메소드가 선언되어 있으면 컴파일 오류 발생

* 매개변수와 리턴값이 없는 람다식

  ```java
  @FunctionalInterface
  public interface MyFunctionalInterface {
    public void method();
  }
  ```

  ...

  * 람다식을 인터페이스 변수에 대입하게 되면 익명 구현 객체가 만들어지게 되고,
  * 실행은 인터페이스 변수를 통해 메소드를 호출해 주면 된다.

  ```java
  Myfunctionalnterface fi = () -> { ... }

  fi.method();
  ```

* 매개변수가 있는 람다식

  ```java
  @FunctionalInterface
  public interface MyFunctionalInterface {
    public void method(int x);
  }
  ```

  ...

  ```java
  Myfunctionalnterface fi = (x) -> { ... }
  또는
  Myfunctionalnterface fi = x -> { ... }

  fi.method(5);
  ```

* 리턴값이 있는 람다식

  ```java
  @FunctionalInterface
  public interface MyFunctionalInterface {
    public int method(int x);
  }
  ```

  ...

  ```java
  Myfunctionalnterface fi = (x, y) -> { ...; return 값; }

  int result = fi.method(5, 2);
  ```

  * 약식 표현

    * 1

    ```java
    Myfunctionalnterface fi = (x, y) -> { 
      return x + y;
    }
    ```

    =

    ```java
    Myfunctionalnterface fi = (x, y) -> x + y;
    ```

    * 2

    ```java
    Myfunctionalnterface fi = (x, y) -> { 
      return sum(x, y);
    }
    ```

    =

    ```java
    Myfunctionalnterface fi = (x, y) -> sum(x, y);
    ```

     

  

## 4절. 클래스 멤버와 로컬 변수 사용

* 클래스의 멤버 사용

  * 람다식 실행 블록에는 클래스의 멤버인 필드와 메소드를 제약 없이 사용할 수 있다.
  * 람다식 실행 블록내에서 this는 람다식을 실행한 객체의 참조이다.

  ```java
  public class ThisExample {
    public int outerField = 10;
    
    class Inner {
      int innerField = 20;
      
      void method() {
        MyfunctionalInterface fi = () -> {
          System.out.println("outerField: " + outerField);
          System.out.println("outerField: " + ThisExample.this.outerField);
          
          System.out.println("innerField: " + innerField);
          System.out.println("innerField: " + this.innerField);
        }
        fi.method();
      }
    }
  }
  ```

* 로컬 변수의 사용

  * 람다식은 함수적 인터페이스의 익명 구현 객체를 생성한다.
  * 람다식에서 사용하는 **외부 로컬 변수는 final 특성을 갖는다.**
    * 원래 자바 익명 구현 객체에서 로컬 변수를 사용하게 되면, 그 로컬 변수는 final 특성을 가지게 된다. 람다도 마찬가지이다. 람다식이 익명 구현 객체를 생성하기 때문이다.
    * 자바 8 이후부터는 로컬 변수에 final 키워드를 붙이지 않아도 자동으로 처리해준다.

  ```java
  public class UsingLocalVariable {
    void method(int arg) {  //arg는 final 특성을 가짐
      int localVar = 40     //localVar는 final 특성을 가짐
        
      ------------------
        //arg = 31;      //final 특성 때문에 수정 불가
        //localVar = 41; //final 특성 때문에 수정 불가
   	------------------
      
      MyFunctionalInterface fi = () -> {
        //로컬변수 사용
        System.out.println("arg: " + arg);
        System.out.println("localVar: " + localVar);
      }
      fi.method();
    }
  }
  ```

  

## 5절. 표준 API의 함수적 인터페이스

* 한 개의 추상 메소드를 가지는 인터페이스들은 모두 람다식 사용 가능

  * ex: Runnable 인터페이스를 구현해서 new Thread(Runnable 구현 객체) 형태로 쓰지 않고
  * new Thread(() -> { …;}) 이런 식으로 람다식으로 작성해서 사용하는 경우가 대부분이다.

  ```java
  Thread thread = new Thread(() -> {
    for (int i = 0; i < 10; i++) {
      System.out.print(i + " ");
    }
    System.out.println();
  });
  thread.start();
  ```

* 자바 8부터 표준 API로 제공되는 함수적 인터페이스

  * java.util.function 패키지에 포함되어 있다.
  * 매개타입으로 사용되어 람다식을 매개값으로 대입할 수 있도록 해준다.
  * 종류
    * **Consumer** 함수적 인터페이스 류(소비)
      * 매개값만 있고 리턴값이 없는 추상 메소드를 가지고 있다.
    * **Supplier** 함수적 인터페이스 류(공급)
      * 매개값은 없고 리턴값만 있는 추상 메소드를 가지고 있다.
    * **Function** 함수적 인터페이스 류
      * 매개값과 리턴값이 모두 있는 추상 메소드를 가지고 있다.
      * 주로 매개값을 리턴값으로 매핑(타입변환)할 경우에 사용
      * 예를 들면, String 10을 Integer 10으로 변환할 경우
    * **Operator** 함수적 인터페이스 류
      * 매개값과 리턴값이 모두 있는 추상메소드를 가지고 있다.
      * 주로 매개값을 연산하고 그 결과를 리턴할 경우에 사용
    * **Predicate** 함수적 인터페이스 류
      * 매개값을 조사해서 true 또는 false를 리턴할 때 사용

#### Consumer 함수적 인터페이스

* Consumer 함수적 인터페이스의 특징은 리턴값이 없는 accept() 메소드를 가지고 있다. accept() 메소드는 단지 매개값을 소비하는 역할만 한다. 여기서 소비한다는 말은 사용만 할 뿐 리턴값이 없다는 뜻이다.

| 인터페이스명                 | 추상 메소드                         | 설명                    |
| ---------------------- | ------------------------------ | --------------------- |
| Consumer\<T\>          | void accept(T t)               | 객체 T를 받아 소비           |
| BiConsumer\<T,U\>      | void accept(T t, U u)          | 객체 T와 U를 받아 소비        |
| DoubleConsumer         | void accept(double value)      | double 값을 받아 소비       |
| IntConsumer            | void accept(int value)         | int 값을 받아 소비          |
| LongConsumer           | void accept(long value)        | long 값을 받아 소비         |
| ObjDoubleConsumer\<T\> | void accept(T t, double value) | 객체 T와 double 값을 받아 소비 |
| ObjIntConsumer\<T\>    | void accept(T t, int value)    | 객체 T와 int 값을 받아 소비    |
| ObjLongConsumer\<T\>   | void accept(T t, long value)   | 객체 T와 long 값을 받아 소비   |

* Consumer\<T\>
  * ```Consumer<String> consumer = t -> {t를 소비하는 실행문;};```
  * \<String\>이므로 매개값 t는 String 타입
* BiConsumer\<T, U\>
  * ```Consumer<String, String> consumer = (t, u) -> {t를 소비하는 실행문;};```
  * \<String, String\>이므로 매개값 t, u는 String 타입
* DoubleConsumer
  - ```DoubleConsumer consumer = d -> {d를 소비하는 실행문;};```
  - 매개값 d는 double 타입(고정)
* ObjIntConsumer\<T\>
  * ```ObjIntConsumer<String> consumer = (t, i) -> { t와 i를 소비하는 실행문; }```
  * \<String\> 이므로 매개값 t는 String 타입이고, 매개값 i는 int 타입(고정)
* Consumer 함수적 인터페이스 예제 코드

```java
import java.util.function.BiConsumer;
import java.util.function.Consumer;
import java.util.function.DoubleConsumer;
import java.util.function.ObjIntConsumer;

public class ConsumerEx {
  public static void main(String[] args) {
    Consumer<String> consumer = t -> System.out.println(t + "8");
    consumer.accept("Java");

    BiConsumer<String, String> biConsumer = (t, u) -> System.out.println(t + u);
    biConsumer.accept("Java", "8");

    DoubleConsumer doubleConsumer = d -> System.out.println("Java" + d);
    doubleConsumer.accept(8.0);

    ObjIntConsumer<String> objIntConsumer = (t, i) -> System.out.println(t + i);
    objIntConsumer.accept("Java", 8);
  }
}
```

```java
Java8
Java8
Java8.0
Java8
```

  

#### Supplier 함수적 인터페이스

* Supplier 함수적 인터페이스의 특징은 매개변수가 없고 리턴값이 있는 getXXX() 메소드를 가지고 있다. 이들 메소드는 실행 후 호출한 곳으로 데이터를 리턴(공급)하는 역할을 한다.
* 리턴 타입에 따라서 아래와 같은 Supplier 함수적 인터페이스들이 있다.

| 인터페이스명          | 추상 메소드                 | 설명            |
| --------------- | ---------------------- | ------------- |
| Supplier\<T\>   | T get()                | 객체를 리턴        |
| BooleanSupplier | boolean getAsBoolean() | boolean 값을 리턴 |
| DoubleSupplier  | double getAsDouble()   | double 값을 리턴  |
| IntSupplier     | int getAsInt()         | int 값을 리턴     |
| LongSupplier    | long getAsLong()       | long 값을 리턴    |

* Supplier\<T\>
  * ```Supplier<String> supplier = () -> {...; return "문자열";}```

- IntSupplier
  - ```IntSupplier supplier = () -> {...; return int값;}```
- Supplier 함수적 인터페이스 예제 코드

```java
import java.util.function.IntSupplier;

public class SupplierEx {
  public static void main(String[] args) {
    IntSupplier intSupplier = () -> {
      int num = (int) (Math.random() * 6) +1;
      return num;
    };
    int num = intSupplier.getAsInt();
    System.out.println("랜덤 주사위 눈의 수 : " + num);
  }
}
```

```
5
```

  

#### Function 함수적 인터페이스

* Function 함수적 인터페이스의 특징은 매개변수와 리턴값이 있는 applyXXX() 메소드를 가지고 있다. 이들 메소드는 매개값을 리턴값으로 매핑(타입 변환) 하는 역할을 한다.
* Ex) String 10 -> Integer 10

| 인터페이스명                | 추상 메소드                          | 설명                  |
| --------------------- | ------------------------------- | ------------------- |
| Function\<T, R\>      | R apply(T t)                    | 객체 T를 객체 R로 매핑      |
| BiFunction\<T, U, R\> | R apply(T, U)                   | 객체 T와 U를 받아서 R로 매핑  |
| DoubleFunction\<R\>   | R apply(double value)           | double을 객체 R로 매핑    |
| IntToDoubleFunction   | double applyAsDouble(int value) | int를 double로 매핑     |
| ToDoubleBiFunction    | double applyAsDouble(T t, U u)  | 객체 T와 U를 double로 매핑 |
| ToDoubleFunction      | double applyAsDouble(T value)   | 객체 T를 double로 매핑    |
| …                     | ...                             | ...                 |

* Function\<T, R\>

  ```java
  Function<Student, String> function = t -> return t.getName();
  또는
  Function<Student, String> function = t -> t.getName();
  ```

  * \<Student, String\> 이므로 매개값 t는 Student 타입이고 리턴값은 String 타입이다.
  * Student 객체를 String으로 매핑한 예제이다.

* ToIntFunction\<T\>

  ```java
  ToIntFunction<Student> function = t - > return t.getScore();
  또는
  ToIntFunction<Student> function = t - > t.getScore();
  ```

  - \<Student\> 이므로 매개값 t는 Student 타입이고 리턴값은 int 타입이다.(고정)
  - Student 객체를 int으로 매핑한 예제이다.

* Function 함수적 인터페이스 예제 코드

  * Student.java

  ```java
  public class Student {
    private String name;
    private int englishScore;
    private int mathScore;

    public Student(String name, int englishScore, int mathScore) {
      this.name = name;
      this.englishScore = englishScore;
      this.mathScore = mathScore;
    }

    public String getName() {
      return name;
    }

    public void setName(String name) {
      this.name = name;
    }

    public int getEnglishScore() {
      return englishScore;
    }

    public void setEnglishScore(int englishScore) {
      this.englishScore = englishScore;
    }

    public int getMathScore() {
      return mathScore;
    }

    public void setMathScore(int mathScore) {
      this.mathScore = mathScore;
    }
  }
  ```

  * FuntionEx1

  ```java
  import java.util.Arrays;
  import java.util.List;
  import java.util.function.Function;
  import java.util.function.ToIntFunction;

  public class FunctionEx1 {
    private static List<Student> list = Arrays.asList(
        new Student("namjune1", 100, 100),
        new Student("namjune2", 99, 99)
    );

    public static void printString(Function<Student, String> function) {
      list.forEach(student -> System.out.print(function.apply(student) + " "));
    }

    public static void printInt(ToIntFunction<Student> function) {
      list.forEach(student -> System.out.print(function.applyAsInt(student) + " "));
    }

    public static void main(String[] args) {
      System.out.println("[학생 이름]");
      printString(t -> t.getName());
      System.out.println();
      System.out.println("[영어 점수]");
      printInt(t -> t.getEnglishScore());
      System.out.println();
      System.out.println("[수학 점수]");
      printInt(t -> t.getMathScore());
    }
  }
  ```

  ```java
  [학생 이름]
  namjune1 namjune2 
  [영어 점수]
  100 99 
  [수학 점수]
  100 99 
  ```

  * FuntionEx2

  ```java
  import java.util.Arrays;
  import java.util.List;
  import java.util.function.ToIntFunction;

  public class FunctionEx2 {
    private static List<Student> list = Arrays.asList(
        new Student("namjune1", 100, 100),
        new Student("namjune2", 99, 99)
    );

    public static double avg(ToIntFunction<Student> function) {
      int sum = 0;
      for (Student student : list) {
        sum += function.applyAsInt(student);
      }
      double avg = (double) sum / list.size();
      return avg;
    }

    public static void main(String[] args) {
      double englishAvg = avg(t -> t.getEnglishScore());
      System.out.println("[영어 평균 점수]");
      System.out.println(englishAvg);

      System.out.println();

      double mathAvg = avg(t -> t.getMathScore());
      System.out.println("[수학 평균 점수]");
      System.out.println(mathAvg);
    }
  }
  ```

  ```java
  [영어 평균 점수]
  99.5

  [수학 평균 점수]
  99.5
  ```

   

#### Operator 함수적 인터페이스

* Operator 함수적 인터페이스의 특징은 Function과 동일하게 매개변수와 리턴값이 있는 applyXXX() 메소드를 가지고 있다. 하지만 이들 메소드는 매개값을 리턴값으로 매핑(타입변환)하는 역할보다는 매개값을 이용해서 연산을 수행한 후 동일한 타입으로 리턴값을 제공하는 역할을 한다.
* ex) 10+5 -> 15

| 인터페이스명               | 추상메소드                                | 설명               |
| -------------------- | ------------------------------------ | ---------------- |
| BinaryOperator\<T\>  | BiFunction\<T,U,R\>의 하위 인터페이스        | T와 U를 연산한 후 R 리턴 |
| UnaryOperator\<T\>   | Function\<T,R\>의 하위 인터페이스            | T를 연산한 후 R 리턴    |
| DoubleBinaryOperator | double applyAsDouble(double, double) | 두 개의 double 연산   |
| DoubleUnaryOperator  | double applyAsDouble(double)         | 한개의 double 연산    |
| IntBinaryOperator    | int applyAsInt(int, int)             | 두 개의 int 연산      |
| IntUnaryOperator     | int applyAsInt(int)                  | 한 개의 int 연산      |
| LongBinaryOperator   | long applyAsLong(long, long)         | 두 개의 long 연산     |
| LongUnaryOperator    | long applyAsLong(long)               | 한 개의 long 연산     |

* IntBinaryOperator

  ```IntBinaryOperator operator = (a,b) -> { ...; return int 값;}```

  * 매개값 a, b는 모두 int 타입이고, 연산 후 리턴값도 int타입

* IntUnaryOperator

  `IntUnaryOperator operator = a -> { ...; return int값; }`

  * 매개값 a는 int 타입이고, 연산 후 리턴값도 int 타입

* Operator 함수적 인터페이스 예제 코드

  ```java
  import java.util.function.IntBinaryOperator;

  public class OperatorEx {
    private static int[] scores = {92, 34, 87};

    private static int maxOrMin(IntBinaryOperator intBinaryOperator) {
      int result = scores[0];
      for (int score : scores) {
        result = intBinaryOperator.applyAsInt(result, score);
      }
      return result;
    }

    public static void main(String[] args) {
      int max = maxOrMin(
          (result, score) -> {
            if (result >= score) return result;
            else return score;
          }
      );
      System.out.println(max);

      int min = maxOrMin(
          (result, score) -> {
            if (result <= score) return result;
            else return score;
          }
      );
      System.out.println(min);
    }
  }
  ```

  ```java
  92
  34
  ```

  

#### Predicate 함수적 인터페이스

* Predicate 함수적 인터페이스의 특징은 매개변수와 boolean 리턴값이 있는 testXXX() 메소드를 가지고 있다. 이들 메소드는 매개값을 조사해서 true 또는 false를 리턴하는 역할을 한다.

| 인터페이스명              | 추상 메소드                     | 설명             |
| ------------------- | -------------------------- | -------------- |
| Predicate\<T\>      | boolean test(T t)          | 객체 T를 조사       |
| BiPredicate\<T, U\> | boolean test(T t, U u)     | 객체 T와 U를 비교 조사 |
| DoublePredicate     | boolean test(double value) | double 값을 조사   |
| IntPredicate        | boolean test(int value)    | int 값을 조사      |
| LongPredicate       | boolean test(long value)   | long 값을 조사     |

* Predicate\<T\>

  ```java
  Predicate<Student> predicate = t -> { return t.getSex().equals("남자")};
  또는
  Preducate<Sturent> predicate = t -> { t.getSex().equals("남자")};
  ```

  * \<Student\> 이므로 매개값 t는 Student 타입이고, 리턴값은 boolean 타입(고정)

* Predicate 함수적 인터페이스 예제 코드

```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Predicate;

public class PredicateEx {
  private static List<Student> list = Arrays.asList(
      new Student("김남준", "남", 100),
      new Student("김남순", "여", 100),
      new Student("김남준1", "남", 90),
      new Student("김남순2", "여", 80)
  );

  public static double avg(Predicate<Student> predicate) {
    int count = 0;
    int sum = 0;
    for (Student student : list) {
      if (predicate.test(student)) {
        sum += student.getScore();
        count++;
      }
    }
    return sum / count;
  }

  public static void main(String[] args) {
    double maleAvg = avg(student -> student.getSex().equals("남"));
    System.out.println("남자 점수 평균: " + maleAvg);

    double femaleAvg = avg(student -> student.getSex().equals("여"));
    System.out.println("여자 점수 평균: " + femaleAvg);
  }
}
```

```java
남자 점수 평균: 95.0
여자 점수 평균: 90.0
```

  

#### andThen()과 conpose() 디폴트 메소드

* 함수적 인터페이스가 가지고 있는 디폴트 메소드이다.
* 두 개의 함수적 인터페이스를 순차적으로 연결해서 실행한다.
* 첫번째 리턴값을 두번째 매개값으로 제공해서 최종 결과값을 리턴한다.
* andThen()과 compose()의 차이점은 어떤 함수적 인터페이스부터 처리하느냐이다.

  

* andThen() 디폴트 메소드
  * 인터페이스 AB의 method를 실행하면
  * 인터페이스 A의 람다식의 결과값을 받아서
  * 인터페이스 B의 매개값으로 전달하고
  * 최종결과를 리턴을 한다.

```java
인터페이스 AB = 인터페이스 A.andThen(인터페이스 B);
최종결과 = 인터페이스 AB.method();
```

* compose() 디폴트 메소드
  * 인터페이스 AB의 method를 실행하면
  * 인터페이스 B의 람다식의 결과값을 받아서
  * 인터페이스 A의 매개값으로 전달하고
  * 최종결과를 리턴한다.

```java
인터페이스 AB = 인터페이스 A.compose(인터페이스 B);
최종결과 = 인터페이스 AB.method();
```

* 함수적 인터페이스가 andThen()과 compose()를 제공하는지를 알아보고 사용하는 것이 좋다.
* andThen()메소드는 거의 제공을하지만 compose()의 경우 아래의 메소드들만 제공한다.
  * compose() 디폴트 메소드를 제공하는 함수적 인터페이스
    * Function\<T, R\>
    * DoubleUnaryOperator
    * IntUnaryOperator
    * LongUnaryOperator

**Consumer의 순차적 연결**

* Consumer 종류의 함수적 인터페이스는 처리 결과를 리턴하지 않기 떄문에 andThen()과 compose() 디폴트 메소드는 함수적 인터페이스의 호출순서만 정한다.
* 사용 예시

```java
Consumer<Member> consumerA = (m) -> {
  System.out.println("consumerA: " + m.getName());
};
```

```java
Consumer<Member> consumerB = (m) -> {
  System.out.println("consumerB: " + m.getId());
};
```

```java
Consumer<Member> consumerAB = consumerA.andThen(consumerB);
consumerAB.accept(new Member("홍길동", "hong", null));
```

```java
실행 결과:
consumerA: 홍길동
consumerB: hong
```

* 예제코드

  * Member.java

  ```java
  public class Member {
    private String name;
    private String id;
    private Address address;

    public Member(String name, String id, Address address) {
      this.name = name;
      this.id = id;
      this.address = address;
    }

    public String getName() {
      return name;
    }

    public String getId() {
      return id;
    }

    public Address getAddress() {
      return address;
    }
  }
  ```

  * Address.java

  ```java
  public class Address {
    private String country;
    private String city;

    public Address(String country, String city) {
      this.country = country;
      this.city = city;
    }

    public String getCountry() {
      return country;
    }

    public String getCity() {
      return city;
    }
  }
  ```

  * ConsumerAndThenEx.java

  ```java
  public class ConsumerAndThenEx {
    public static void main(String[] args) {
      Consumer<Member> consumerA = (member) -> {
        System.out.println(member.getName());
      };
      Consumer<Member> consumerB = (member) -> {
        System.out.println(member.getId());
      };

      Consumer<Member> consumerAB = consumerA.andThen(consumerB);
      consumerAB.accept(
          new Member("김남준", "njkim", new Address("korea", "seoul"))
      );
    }
  }
  ```

  ```java
  김남준
  njkim
  ```

  

**Function의 순차적 연결**

* Function과 Operator 종류의 함수적 인터페이스는 먼저 실행한 함수적 인터페이스의 결과를 다음 함수적 인터페이스의 매개값으로 넘겨주고, 최종 처리결과를 리턴한다.
* functionA의 두번째 매개변수 A는 functionB로 전달되는 매개값의 타입이고, functionB의 첫번째 매개변수로 정의되어 있다.
* Address는 functionA에서 functionB로 전달되는 타입이고, 최종적으로 functionAB에서는 Member객체를 이용하여 String 타입의 결과값을 얻는다.
* 사용 예시

```java
Function<Member, Address> functionA;
Function<Address, String> functionB;
Function<Member, String> functionAB;
String city;

functionA = m -> m.getAddress();
functionB = a -> a.getCity;
```

```java
functionAB = functionA.andThen(functionB);
city = functionAB.apply(
  new Member("홍길동","hong",new Address("한국", "서울"))
);

functionAB = functionB.compose(functionA);
city = functionAB.apply(
  new Member("홍길동","hong",new Address("한국", "서울"))
);
```

* 예제 코드

  * FunctionAndThenComposeEx

  ```java
  public class FunctionAndThenComposeEx {
    public static void main(String[] args) {
      Function<Member, Address> functionA;
      Function<Address, String> functionB;
      Function<Member, String> functionAB;

      functionA = member -> member.getAddress();
      functionB = address -> address.getCity();

      functionAB = functionA.andThen(functionB);
      String city = functionAB.apply(
          new Member("김남준", "njkim", new Address("한국", "서울"))
      );
      System.out.println("andThen 거주도시: " + city);

      functionAB = functionB.compose(functionA);
      city = functionAB.apply(
          new Member("김남준", "njkim", new Address("한국", "서울"))
      );
      System.out.println("compose 거주도시: " + city);
    }
  }
  ```

  ```java
  andThen 거주도시: 서울
  compose 거주도시: 서울
  ```

  

#### Predicate 함수적 인터페이스의 and(), or(), negate() 디폴트 메소드와 isEqual() 정적 메소드

* and(), or(), negate() 디폴트 메소드
  * Predicate 함수적 인터페이스의 디폴트 메소드
* and() : &&과 대응 - 두 Predicate가 모두 true를 리턴하면 최종적으로 true를 리턴
  * `predicateAB = predicateA.and(predicateB);`
* or() : ||과 대응 - 두 Predicate중 하나만 true를 리턴하면 최종적으로 true를 리턴
  * `predicateAB = predicate.or(predicateB);`
* negate() : !과 대응 - Predicate의 결과가 true리면 false, false이면 true를 리턴
  * `predicateAB = predicateA.negate();`
* and(), or(), negate() 예제 코드

```java
import java.util.function.IntPredicate;

public class PredicateAndOrNegateEx {
  public static void main(String[] args) {
    IntPredicate predicateA = a -> a % 2 == 0;
    IntPredicate predicateB = b -> b % 3 == 0;
    IntPredicate predicateAB = predicateA.and(predicateB);

    int input = 6;
    boolean result = predicateAB.test(input);
    System.out.println(input + "은/는 2의 배수이면서 3의 배수인가? " + result);

    input = 4;
    predicateAB = predicateA.or(predicateB);
    result = predicateAB.test(input);
    System.out.println(input + "은/는 2의 배수 또는 3의 배수 인가? " + result);

    input = 4;
    predicateAB = predicateA.negate();
    result = predicateAB.test(input);
    System.out.println(input + "은/는 2의 배수가 아닌가? " + result);
  }
}
```

```java
6은/는 2의 배수이면서 3의 배수인가? true
4은/는 2의 배수 또는 3의 배수 인가? true
4은/는 2의 배수가 아닌가? false
```

* isEqual() 정적 메소드

  * Predicate\<T\>의 정적 메소드

  ```java
  Predicate<Object> predicate = Predicate.isEqual(targetObject);
  boolean result = predicate.test(sourceObject);
  ```

  * Objects.equals(sourceObject, targetObject)는 다음과 같은 리턴값을 제공한다.

  | sourceObject | targetObject | 리턴값                                    |
  | ------------ | ------------ | -------------------------------------- |
  | null         | null         | true(주의 해야 한다.)                        |
  | not null     | null         | false                                  |
  | null         | not null     | false                                  |
  | not null     | not null     | sourceObject.equals(targetObject)의 리턴값 |

* isEqual() 예제 코드

```java
import java.util.function.Predicate;

public class PredicateIsEqualEx {
  public static void main(String[] args) {
    Predicate<String> predicate;

    predicate = Predicate.isEqual(null);
    System.out.println("null, null: " + predicate.test(null));

    predicate = Predicate.isEqual("java8");
    System.out.println("null, java8: " + predicate.test(null));

    predicate = Predicate.isEqual(null);
    System.out.println("java8, null: " + predicate.test("java8"));

    predicate = Predicate.isEqual("java8");
    System.out.println("java8, java8: " + predicate.test("java8"));
  }
}
```

```java
null, null: true
null, java8: false
java8, null: false
java8, java8: true
```

  

#### minBy(), maxBy() 정적 메소드

* BinaryOperator\<T\> 함수적 인터페이스의 정적 메소드
* Comparator를 이용해서 최대 T와 최소 T를 얻는 BinaryOperator\<T\>를 리턴한다.

| 리턴타입                | 정적 메소드                                  |
| ------------------- | --------------------------------------- |
| BinaryOperator\<T\> | minBy(Comparator<? super T> comperator) |
| BinaryOperator\<T\> | maxBy(Comparator<? super T> comparator) |

* Comparator\<T\>는 다음과 같이 선언된 함수적 인터페이스이다. o1과 o2를 비교해서 o1이 작으면 음수를, 동일하면 0, o1이 크면 양수를 리턴해야하는 compare() 메소드가 선언되어 있다.

```java
@FunctionalInterface
public interface Comparator<T> {
  public int compare(T o1, T o2);
}
```

* Comparator\<T\>를 타겟 타입으로하는 람다식은 다음과 같이 작성할 수 있다.

```java
(o1, o2) -> { ...; return int 값; }
```

* 만약 o1과 o2가 int 타입이라면 다음과 같이 Integer.compare(int, int) 메소드를 이용할 수 있다.
  * 만약 int 타입이 아니라면 직접 비교해서 return 하도록 작성을 해야한다.

```java
(o1. o2) -> Integer.compare(o1, o2);
```

* 예제 코드

```java
import java.util.function.BinaryOperator;

public class OperatorMinByMaxByEx {
  public static void main(String[] args) {
    BinaryOperator<Fruit> binaryOperator;
    Fruit fruit;

    binaryOperator = BinaryOperator.minBy(
        (f1, f2) -> Integer.compare(f1.getPrice(), f2.getPrice()));
    fruit = binaryOperator.apply(
        new Fruit("수박", 20000), new Fruit("오렌지", 10000));
    System.out.println("minBy: " + fruit.getName());

    binaryOperator = BinaryOperator.maxBy(
        (f1, f2) -> Integer.compare(f1.getPrice(), f2.getPrice()));
    fruit = binaryOperator.apply(
        new Fruit("수박", 20000), new Fruit("오렌지", 10000));
    System.out.println("maxBy: " + fruit.getName());
  }
```

```java
minBy: 오렌지
maxBy: 수박
```

