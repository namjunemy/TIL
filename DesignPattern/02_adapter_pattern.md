# 2. Adapter Pattern

> [Code Repository](https://github.com/namjunemy/design_pattern)

연관성 없는 두 객체 묶어 사용하기

### 학습목표

* 알고리즘을 요구사항에 맞춰 사용할 수 있다.

### Adapter

* 사전적의미
  * 기계나 기구등을 다목적으로 사용하기 위한 부가 기구

### 기본 설계

* Adaptee에 정의된 기능을 Adapter를 통해서 요구사항에 맞춰 변경하여 사용한다.

![](https://github.com/namjunemy/TIL/blob/master/DesignPattern/img/adapter_01.png?raw=true)

## 요구 사항

* 두 수에 대한 다음 연산을 수행하는 객체를 만들어 주세요.
  * 수의 두 배의 수를 반환 - twiceOf(Float):Float
  * 수의 반(1/2)의 수를 반환 - halfOf(Float):Float
* 구현 객체 이름은 'Adapter'으로 해주세요.
* Math 클래스에서 두 배와 절반을 구하는 함수는 **이미 구현**되어 있습니다. 매개변수와 리턴타입은 double형.

### 설계

![](https://github.com/namjunemy/TIL/blob/master/DesignPattern/img/adapter_02.png?raw=true)

### 구현

* 이미 구현되어있는 Math클래스의 메소드를 사용하여 twiceOf(),halfOf()를 만드는 요구사항이다.
* Math클래스에 구현되어있는 함수들은 리턴타입과 매개타입이 double형으로 직접사용할 수 없다.
* 선언과 구현을 분리하기 위해 만든 Adapter 인터페이스에서 요구 사항의 메소드를 정의하고,(Float 타입)
* AdapterImpl 구현체에서 Math클래스의 함수들을 Float형으로 변환해서 리턴해주는 작업을 통해 요구사항을 만족시킬 수 있다. 

```java
public class Math {
  public static double twoTime(double num) {
    return num * 2;
  }

  public static double half(double num) {
    return num / 2;
  }

  public static Double doubled(Double d) {
    return d * 2;
  }
}
```

```java
public interface Adapter {
  public Float twiceOf(Float num);
  public Float halfOf(Float num);
}
```

```java
public class AdapterImpl implements Adapter {
  @Override
  public Float twiceOf(Float num) {
    return (float) Math.twoTime(num.doubleValue());
  }

  @Override
  public Float halfOf(Float num) {
    return (float) Math.half(num.doubleValue());
  }
}
```

```java
public class Main {
  public static void main(String[] args) {
    Adapter adapter = new AdapterImpl();

    System.out.println(adapter.twiceOf(100F));
    System.out.println(adapter.halfOf(88F));
  }
}
```

  

## 실습

* 알고리즘 변경을 원합니다.
  * Math 클래스에 새롭게 두 배를 구할 수 있는 함수가 추가되었습니다. 새로 구현된 알고리즘을 이용하도록 프로그램을 수정해주세요.
* 절반을 구하는 기능에서 로그를 찍는 기능을 추가해 주시기 바랍니다.

### 구현

* Adapter클래스와 Main클래스의 수정없이, 구현체인 AdapterImpl 클래스 코드 수정만으로 새로운 요구 사항을 반영할 수 있다.

```java
public class AdapterImpl implements Adapter {
  @Override
  public Float twiceOf(Float num) {
    return Math.doubled(num.doubleValue()).floatValue();
  }

  @Override
  public Float halfOf(Float num) {
    System.out.println("halfOf() 호출");
    return (float) Math.half(num.doubleValue());
  }
}
```



## 응용

* List로 주어진 데이터를 어떤 복잡한 알고리즘으로 계산한 결과를 반환하는 메소드를 정의하시오. 라는 요구사항이 있고,
* 해당 요구사항을 해결하기 위해 주어진 복잡한 알고리즘이 array로 구현 되어 있다고 가정하자.
* 위의 요구사항을 만족하는 메소드를 구현하기 위해서 Adapter Pattern이 사용될 수 있다.
* Adapter 구현체의 메소드에서 List 데이터를 array로 변환하고, 주어진 알고리즘을 이용하여 연산한 뒤, 다시 그 결과를 List데이터로 변환하여 결과를 리턴한다. 이것은 요구사항을 만족시킨 메소드이다.