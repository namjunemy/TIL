# 7. Builder Pattern

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







