# 6. Prototype Pattern

> 출처 : Head First Design Patterns & 인프런 - '자바 디자인 패턴의 이해'(GoF Design Pattern)
>
> [Code Repository](https://github.com/namjunemy/design_pattern)

"생산 비용 줄이기"

### 사전적 의미의 Prototype 이란?

- 원형; 본, 표준, 모범
- (어떤 것의)옛날의 유사물
- 원형(archetype)

### 학습 목표

* 프로토 타입 패턴을 통해서 복잡한 인스턴스를 복사 할 수 있다.
* 생산 비용이 높은 인스턴스를 복사를 통해서 쉽게 생성할 수 있도록 하는 패턴

### 인스턴스 생산 비용이 높은 경우

* 종류가 너무 많이서 클래스로 정리되지 않는 경우
* 클래스로부터 인스턴스 생성이 어려운 경우(그래픽 에디터에서 사용자의 마우스 클릭으로 생성되는 객체들)

### 기본 설계

* 프로토타입 인스턴스를 등록해 놓고, 등록된 인스턴스를 복사해서 인스턴스를 생성한다.

![](https://github.com/namjunemy/TIL/blob/master/DesignPattern/img/prototype_01.png?raw=true)

### 요구 사항

* 일러스트레이터와 같은 그림 그리기 툴을 개발중입니다. 어떤 모양을 그릴 수 있도록 하고, 복사 붙여넣기 기능을 구현해주세요.

### 구현

* Java에서는 Colneable 인터페이스를 통해서 clone()을 지원한다.
* Shape 이라는 Cloneable 인터페이스를 구현한 클래스를 만들고,
* 실제로 Shape를 상속받은 클래스에서 clone()이 이루어지는 copy() 메소드를 통해서 인스턴스의 복사가 이루어 진다.
* copy()가 이루어질때는 Circle이 겹치지 않도록 x, y좌표를 +1 해준다.

```java
public class Shape implements Cloneable {
  private String id;

  public String getId() {
    return id;
  }

  public void setId(String id) {
    this.id = id;
  }
}
```

```java
public class Circle extends Shape{
  private int x, y, r;

  public Circle(int x, int y, int r) {
    this.x = x;
    this.y = y;
    this.r = r;
  }

  public int getX() {
    return x;
  }

  public int getY() {
    return y;
  }

  public int getR() {
    return r;
  }

  public Circle copy() throws CloneNotSupportedException {
    Circle circle = (Circle) clone();
    circle.x += 1;
    circle.y += 1;
    return circle;
  }
}
```

```java
public class Main {
  public static void main(String[] args) throws CloneNotSupportedException {
    Circle circle1 = new Circle(1, 1, 3);
    Circle circle2 = circle1.copy();

    System.out.println(circle1.getX() + "," + circle1.getY() + "," + circle1.getR());
    System.out.println(circle2.getX() + "," + circle2.getY() + "," + circle2.getR());
  }
}
```

## Deep Copy VS Shallow Copy

* Shallow Copy
  * 자바의 argument 전달방식은 call-by-value이다. 따라서, reference type에 값을 복사하거나 할당하게 되면 reference 값만을 복사하게 되어 같은 참조를 가진 두개의 변수가 생긴다.

```java
package deepshallow;

public class Cat {
  private String name;

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }
}
```

```java
package deepshallow;

public class Main {
  public static void main(String[] args) {
    Cat navi = new Cat();
    navi.setName("navi");

    Cat yo = navi;
    yo.setName("yo");

    System.out.println(navi.getName());
    System.out.println(yo.getName());
  }
}
```

```shell
yo
yo

Process finished with exit code 0
```

* Deep Copy
  * Clonable 인터페이스를 구현상속해서, clone()을 통한 Deep Copy를 할 수 있다.

```java
package deepshallow;

public class Cat implements Cloneable {
  private String name;

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public Cat copy() throws CloneNotSupportedException{
    Cat cat = (Cat) clone();
    return cat;
  }
}
```

```java
package deepshallow;

public class Main {
  public static void main(String[] args) throws CloneNotSupportedException {
    Cat navi = new Cat();
    navi.setName("navi");

    Cat yo = navi.copy();
    yo.setName("yo");

    System.out.println(navi.getName());
    System.out.println(yo.getName());
  }
}
```

```shell
navi
yo

Process finished with exit code 0
```

* **주의할 점**
  * Cloneable 인터페이스를 구현한 객체를 clone()할 때, 객체 내부의 멤버중 reference type이 있다면, 이것은 **Shallow Copy가 이루어 진다**. 다음 코드의 결과를 보자.

```java
package deepshallow;

public class Cat implements Cloneable {
  private String name;
  private Age age;

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public Age getAge() {
    return age;
  }

  public void setAge(Age age) {
    this.age = age;
  }

  public Cat copy() throws CloneNotSupportedException{
    Cat cat = (Cat) clone();
    return cat;
  }
}
```

```java
package deepshallow;

public class Main {
  public static void main(String[] args) throws CloneNotSupportedException {
    Cat navi = new Cat();
    navi.setName("navi");
    navi.setAge(new Age(2000, 3));

    Cat yo = navi.copy();
    yo.setName("yo");
    yo.getAge().setYear(2010);
    yo.getAge().setValue(7);

    System.out.println(navi.getName());
    System.out.println(navi.getAge().getYear() + "," + navi.getAge().getValue());

    System.out.println(yo.getName());
    System.out.println(yo.getAge().getYear() + "," + yo.getAge().getValue());
  }
}
```

```shell
navi
2010,7
yo
2010,7

Process finished with exit code 0
```

* 따라서, Cloneable 인터페이스를 구현한 객체를 clone()할 때, reference type 멤버도 Cloneable 인터페이스를 구현한 객체로 선언하고, 명시적으로 Deep Copy를 해줘야 한다.
  * `cat.setAge((Age) age.clone());`

```java
package deepshallow;

public class Age implements Cloneable{
  private int year;
  private int value;

  public Age(int year, int value) {
    this.year = year;
    this.value = value;
  }

  public int getYear() {
    return year;
  }

  public void setYear(int year) {
    this.year = year;
  }

  public int getValue() {
    return value;
  }

  public void setValue(int value) {
    this.value = value;
  }

  @Override
  public Object clone() throws CloneNotSupportedException {
    return super.clone();
  }
}
```

```java
package deepshallow;

public class Cat implements Cloneable {
 
  ...
      
  public Cat copy() throws CloneNotSupportedException {
    Cat cat = (Cat) clone();
    cat.setAge((Age) age.clone());
    return cat;
  }
}
```

```java
public class Main {
  public static void main(String[] args) throws CloneNotSupportedException {
    Cat navi = new Cat();
    navi.setName("navi");
    navi.setAge(new Age(2000, 3));

    Cat yo = navi.copy();
    yo.setName("yo");
    yo.getAge().setYear(2010);
    yo.getAge().setValue(7);

    System.out.println(navi.getName());
    System.out.println(navi.getAge().getYear() + "," + navi.getAge().getValue());

    System.out.println(yo.getName());
    System.out.println(yo.getAge().getYear() + "," + yo.getAge().getValue());
  }
}
```

```shell
navi
2000,3
yo
2010,7

Process finished with exit code 0
```

