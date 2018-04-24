# 7. Builder Pattern 01

> 출처 : Head First Design Patterns & 인프런 - '자바 디자인 패턴의 이해'(GoF Design Pattern)
>
> [Code Repository](https://github.com/namjunemy/design_pattern)

"복잡한 단계가 있는 인스턴스 생성과정 단순화"

### 학습목표

* 복잡한 단계가 필요한 인스턴스 생성을 빌더 패턴을 통해서 구현할 수 있다.
* 키워드
  * **복잡한 단계**

### 사전적 의미의 Builder란?

* 건축 업자, 시공자, 건조자
* (새 국가 등의)건설자

### Builder Pattern

* 복잡한 단계를 거쳐야 생성되는 객체의 구현을 서브 클래스에게 넘겨주는 패턴

### 기본 설계

* 기본적인 설계는 Director가 Builder를 가지고 build하는 패턴이지만, 정확히 이 설계가 아닌 다양한 방식으로 만들어 질 수 있다.
* 실제로 구현하는 단계에서는 ComputerFactory가 BluePrint 인터페이스를 구현한 각 제조사별 BluePrint로 컴퓨터를 만들어 내는 코드를 작성해 본다.

![](https://github.com/namjunemy/TIL/blob/master/DesignPattern/img/builder_01.png?raw=true)

### 구현

* Director - ComputerFactory
* AbstractBuilder - BluePrint
* ConcreteBuilder - MacBluePrint
* ComputerFactory.java
  * 컴퓨터를 만들어내는 공장 클래스이다.
  * 설계도(BluePrint)를 가지고 cpu, ram, storage를 조립하고 만들어진 컴퓨터를 반환한다.

```java
public class ComputerFactory {
  private BluePrint bluePrint;

  public void setBluePrint(BluePrint bluePrint) {
    this.bluePrint = bluePrint;
  }

  public void make() {
    bluePrint.setCpu();
    bluePrint.setRam();
    bluePrint.setStorage();
  }

  public Computer get() {
    return bluePrint.getComputer();
  }
}
```
* Computer.java
  * 컴퓨터 클래스
```java
public class Computer {
  private String cpu;
  private String ram;
  private String storage;

  public Computer(String cpu, String ram, String storage) {
    this.cpu = cpu;
    this.ram = ram;
    this.storage = storage;
  }

  public String getCpu() {
    return cpu;
  }

  public void setCpu(String cpu) {
    this.cpu = cpu;
  }

  public String getRam() {
    return ram;
  }

  public void setRam(String ram) {
    this.ram = ram;
  }

  public String getStorage() {
    return storage;
  }

  public void setStorage(String storage) {
    this.storage = storage;
  }

  @Override
  public String toString() {
    StringBuilder sb = new StringBuilder();
    sb.append("cpu : ").append(cpu);
    sb.append("\nram : ").append(ram);
    sb.append("\nstorage : ").append(storage);
    return sb.toString();
  }
}
```
* BluePrint.java
  * 설계도의 추상클래스이다.
  * 각요소를 set하고, 만들어진 Computer를 return 한다.

```java
public abstract class BluePrint {
  public abstract void setCpu();

  public abstract void setRam();

  public abstract void setStorage();

  public abstract Computer getComputer();
}
```
* MacBluePrint.java
  * AbstractBuilder인 BluePrint 클래스의 ConcreteBuilder 클래스이다.
```java
public class MacBluePrint extends BluePrint {
  private Computer computer;

  public MacBluePrint() {
    computer = new Computer("default", "default", "default");
  }

  @Override
  public void setCpu() {
    computer.setCpu("i5");
  }

  @Override
  public void setRam() {
    computer.setCpu("8gb");
  }

  @Override
  public void setStorage() {
    computer.setStorage("ssd-250gb");
  }

  @Override
  public Computer getComputer() {
    return this.computer;
  }
}
```
* Main.java
```java
public class Main {
  public static void main(String[] args) {
    ComputerFactory computerFactory = new ComputerFactory();
    computerFactory.setBluePrint(new MacBluePrint());
    computerFactory.make();
    Computer computer = computerFactory.get();
    System.out.println(computer);
  }
}
```

* 복잡하게 생성되어야 하는 객체의 구현을 서브클래스에게 넘겨서 단순화 시킬 수 있다.

# Builder Pattern 02

실제로 현업에서 쓰이는 빌더 패턴이 무엇인지 알아 본다.

### 학습 목표

* 많은 변수를 가진 객체의 생성을 가독성 높게 코딩 할 수 있다.

### 키워드

* 가독성 & 많은 멤버 변수

### Builder Pattern

* 많은 인자를 가진 객체의 생성을 다른 객체의 도움으로 생성하는 패턴

### 구현

```java
package pattern02;

public class Computer {
  private String cpu;
  private String ram;
  private String storage;

  public Computer(String cpu, String ram, String storage) {
    this.cpu = cpu;
    this.ram = ram;
    this.storage = storage;
  }

  public String getCpu() {
    return cpu;
  }

  public void setCpu(String cpu) {
    this.cpu = cpu;
  }

  public String getRam() {
    return ram;
  }

  public void setRam(String ram) {
    this.ram = ram;
  }

  public String getStorage() {
    return storage;
  }

  public void setStorage(String storage) {
    this.storage = storage;
  }

  @Override
  public String toString() {
    StringBuilder sb = new StringBuilder();
    sb.append("cpu : ").append(cpu);
    sb.append("\nram : ").append(ram);
    sb.append("\nstorage : ").append(storage);
    return sb.toString();
  }
}
```

```java
package pattern02;

public class ComputerBuilder {
  private Computer computer;

  public ComputerBuilder() {
    computer = new Computer("default","default","default");
  }

  public static ComputerBuilder start() {
    return new ComputerBuilder();
  }

  public ComputerBuilder setCpu(String cpu) {
    computer.setCpu(cpu);
    return this;
  }

  public ComputerBuilder setRam(String ram) {
    computer.setRam(ram);
    return this;
  }

  public ComputerBuilder setStorage(String storage) {
    computer.setStorage(storage);
    return this;
  }

  public Computer build() {
    return this.computer;
  }
}
```

```java
package pattern02;

public class Main {
  public static void main(String[] args) {
    Computer computer = ComputerBuilder
        .start()
        .setCpu("i5")
        .setRam("8gb")
        .setStorage("256 ssd")
        .build();

    System.out.println(computer);
  }
}
```



