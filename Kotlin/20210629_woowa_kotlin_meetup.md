# 우아한 Meet-Up 코틀린

> 사내 코틀린 밋업을 정리한 내용

## 처음 자바에서 코틀린으로 변경하고 느꼈던 점

> 라이브 선물 스쿼드/박정운님(안드로이드 개발자)

### 1. Null을 다시 생각하게 됨

NPE를 유발시키는 Nullable 객체가 자바에서는 자연스러웠지만 

코틀린에서는 기본적으로 Null을 허용하지 않기 때문에 

변수나 객체 생성시 Nullable 여부에 대하여 다시 한번 고민하게 된다.

- kotlin

  ```kotlin
  val name: String = null // Error
  val name: String? = null
  ```

### 2. 더이상 '+'를 안써도 괜찮아

문자열 결합시 사용하던 '+'(또는 StringBuilder)를

코틀린에서는 언어차원에서 문자열에 $로 변수를 할당해서 문자열 결합을 할 수 있다.

- kotlin

  ```kotlin
  val name = "NJ"
  val person = Person("NJ")
  
  println("Hello $name")
  println("Hello ${person.name}")
  
  data class Person(
    val name: String
  )
  ```

### 3. 더 이상 'if (object != null)'를 안써도 괜찮아

코틀린에서는 Safe Call 이라고 불리는 `.`(dot) 을 이용해서 null 체크를 하는 기능이 있다.

코드에서 변수의 `?.` 다음 코드블럭은 null이 아닌 경우에만 진행이 되고

변수가 null인 경우에는 Elvis Operation 이라고 불리는 `?:` 을 통해서 처리할 수 있다.

아래와 같이 단계적으로 null 체크가 들어가야 하는 상황에서 Safe Call 기능을 통해 체이닝으로 처리할 수도 있다!

- java

  ```java
  if (person != null) {
    if (person.name != null) {
      return true
    } else {
      return false
    }
  } else {
    return false
  }
  ```

- kotlin

  - Safe Call 체이닝중 하나가 null일 경우 false가 반환된다

  ```kotlin
  val data = person?.name?.let {
    true
  } ?: false
  ```

### 4. switch 안녕~ 반가워 when!

switch-case 구문 보다 간결하게 분기 처리를 할 수 있다.

- java

  ```java
  switch (code) {
    case 1:
      do1();
      break;
    case 2:
      do2();
      break;
    case 3:
      do3();
      break;
    default:
      doDefault();
      break;
  }
  ```

- kotlin 

  ```kotlin
  val code = 3
  
  when (code) {
    1 -> do1()
    2 -> do2()
    3 -> do3()
    else -> doDefault()
  }
  ```

### 5. default 값을 사용할 수가 있다

코틀린에서는 함수의 파라미터에 기본값을 넣을 수 있다.

따라서, 파라미터가 있는 함수를 아규먼트 없이 호출도 가능하다!

```kotlin
fun testMethod(code: Int = 123) {
  ...
}

testMethod()     //Ok
testMethod(321)  //Ok
```

### 6. 더 이상 getter/setter 안 만들어서 좋구나

코틀린의 data 클래스는 getter, setter, toString, equalsAndHashCode, constructor를 포함한다.

```kotlin
data class Person(
  val name: String,
  val age: Int,
  val phoneNumber: String,
  val address: String
)
```

 ### 7. singleton도 간단하게 object로!

자바에서 별도의 코드로 작성했던 싱글톤 클래스를 코틀린에서는 object 키워드를 사용하여 간단하게 사용할 수 있다.

- java

  ```java
  public class Singleton {
    private Singleton() {
      
    }
    
    public static Singleton getInstance() {
      return LazyHolder.INSTANCE;
    }
    
    private static class LazyHolder {
      private static final Singleton INSTANCE = new Singleton();
    }
  }
  ```

- kotlin

  ```kotlin
  object Singleton {
    
  }
  ```

### 8. Type 체크도 간단하게 when-is/else로

자바에서 instanceof로 객체의 타입을 체크 했던 부분을 when-is/else로 간단히 처리할 수 있다.

(코틀린의 Any는 자바의 Object와 다르다. 학습해보자)

- java

  ```java
  void testMethod(Object obj) {
    if (obj instanceof String) {
      doString();
    } else if (obj instanceof Integer) {
      doInteger();
    } else if (obj instanceof Double) {
      doDouble();
    } else {
      doDefault();
    }
  }
  ```

- kotlin

  ```kotlin
  fun testMethod(obj: Any) {
    when (obj) {
      is String -> doString()
      is Int -> doInt()
      is Double -> doDouble()
      else -> doDefault()
    }
  }
  ```

### 9. Mutable과 Immutable에 대해서 다시 한번 고민

자바에서도 final이 있지만, 코틀린에서는 선언으로 변수, 객체 생성에 지정해야하기 때문에 한 번 더 고민할 수 있다.

val = immutable, var mutable

```kotlin
private val immutableObject: ImmutableObject
private var mutablePoint: Int = 0
```

### 10. 확장 함수는 너무 좋다

확장 함수는 특정 자료형에서 바로 특정 함수를 사용할 수 있는 기능이다.

```kotlin
private fun Int myPlus(number: Int) = this + number


12.myPlus(32)
```

### 11. 상속 및 implements가 직관적이다

아래와 같이 `,` 로 상속과 implements를 표현할 수 있다. 하지만, 자바와 마찬가지로 다중상속을 허용하지 않아서 class 2개를 상속 받으면 에러난다.

```kotlin
class TeslaService : CarService(), MyInterface1, MyInterface2
```

### 12. 함수 아규먼트에 이름을 줄 수 있다

함수 아규먼트에 이름을 줄 수 있어서 다른 값이 할당되는 것을 방지할 수 있다.

```kotlin
data class Person(
  val name: String,
  val age: Int,
  val phoneNumber: String,
  val address: String
)

val jungwoon = Person(
  age = 29,
  name = "NJ",
  address = "Korea",
  phoneNumber = "01012341234"
)
```











