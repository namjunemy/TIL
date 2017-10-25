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

  ​

