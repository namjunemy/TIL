# 1. Strategy Pattern

> 출처 : Head First Design Patterns & 인프런 - '자바 디자인 패턴의 이해'(GoF Design Pattern)
>
> [Code Repository](https://github.com/namjunemy/design_pattern)

**"전략 바꾸기"**

### 학습목표

* interface 개념을 이해한다.
* delegate 개념을 이해한다.
* Strategy Pattern 개념을 이해한다.

## 인터페이스

* 사전적 의미
  * 키보드나 디스플레이 따위처럼 사람과 컴퓨터를 연결하는 장치


* Java
  * 기능에 대한 선언과 구현 분리
    * 해당 기능에 대해 선언하는 인터페이스와
    * 그 인터페이스를 implement하여 기능을 구현하는 클래스 
  * 기능을 사용하는 통로
    * 기능을 구현한 클래스의 메소드를 사용할 수 있도록 접점의 역할을 함
* 예제

```java
public interface Ainterface {
  public void funcA();
}
```

```java
public class AinterfaceImpl implements Ainterface{
  @Override
  public void funcA() {
    System.out.println("AAA");
  }
}
```

```java
public class Main {
  public static void main(String[] args) {
    Ainterface ainterface = new AinterfaceImpl();
    ainterface.funcA();
  }
}
```

## Delegate

* 위임하다.
* 어떤 기능을 구현할 때, 그 책임을 다른 객체로 넘기는 것. 다른 객체의 기능을 빌려 사용하는 것을 delegate, 위임한다고 한다.
* 예제

```java
public class AObject {
  Ainterface ainterface;

  public AObject() {
    this.ainterface = new AinterfaceImpl();
  }

  public void funcAA() {
    ainterface.funcA();
    ainterface.funcA();
  }
}
```

 ```java
public class Main {
  public static void main(String[] args) {
    AObject aObject = new AObject();
    aObject.funcAA();
  }
}
 ```

## Strategy Pattern

* 여러 알고리즘을 하나의 추상적인 접근점을 만들어 접근 점에서 서로 교환 가능하도록 하는 패턴.

* 기본 설계

  * 전략을 사용하는 Client가 존재하고, 실제로 전략을 하나 소유하고 있다.
  * Strategy 인터페이스를 구현상속한 각각의 A, B, C전략을 Client가 유연하게 바꿔가면서 사용할 수 있다.

  ![](https://github.com/namjunemy/TIL/blob/master/DesignPattern/img/strategy_01.png?raw=true)

### 요구사항

* 신작 게임에서 캐릭터와 무기를 구현해보세요.
* 무기는 두가지 종류가 있다.
  * 칼, 검
* Weapon 인터페이스는 칼과 검을 사용할 수 있는 접접이 된다. 동시에 attack() 메소드의 구현을 무기마다 분리한다. 
* setter를 통해 무기의 교환이 가능하도록 한다.
* attack() 메소드를 정의하고, delegate를 통해 기능을 위임한다.
* **설계 및 구현**

![](https://github.com/namjunemy/TIL/blob/master/DesignPattern/img/strategy_02.png?raw=true)

```java
public interface Weapon {
  public void attack();
}
```

```java
public class Knife implements Weapon{
  @Override
  public void attack() {
    System.out.println("칼 공격");
  }
}
```

```java
public class Sword implements Weapon{
  @Override
  public void attack() {
    System.out.println("검 공격");
  }
}
```

```java
public class Character {
  private Weapon weapon;

  public void setWeapon(Weapon weapon) {
    this.weapon = weapon;
  }

  public void attack() {
    if (weapon == null)
      System.out.println("맨손 공격");
    else
      weapon.attack();
  }
}
```

```java
public class Main {
  public static void main(String[] args) {
    Character character = new Character();
    character.attack();

    character.setWeapon(new Knife());
    character.attack();

    character.setWeapon(new Sword());
    character.attack();
  }
}
```

### 유지 보수 요청

* 무기 중 도끼를 추가해 주세요.
* 다른 코드의 변경 없이, Ax를 추가하고 효율적으로 사용할 수 있다.

```java
public class Ax implements Weapon{
  @Override
  public void attack() {
    System.out.println("도끼 공격");
  }
}
```

```java
public class Main {
  public static void main(String[] args) {
    Character character = new Character();
    character.attack();

    character.setWeapon(new Knife());
    character.attack();

    character.setWeapon(new Sword());
    character.attack();
    
    character.setWeapon(new Ax());
    character.attack();
  }
}
```

