# 8. Abstract Factory Pattern

> 출처 : Head First Design Patterns & 인프런 - '자바 디자인 패턴의 이해'(GoF Design Pattern)
>
> [Code Repository](https://github.com/namjunemy/design_pattern)

"객체 생성의 가상화"

### 학습목표

* 관련 있는 객체의 생성을 가상화 할 수 있다.
* 키워드
  * 생성 부분의 가상화
  * 관련있는 객체

### 기본 설계

* 생성하는 부분인 Factory를 가상화해서 ConcreteFactory 클래스를 가려주고,
* Client는 가상화된 Factory와 Product를 활용한다.

![](https://github.com/namjunemy/TIL/blob/master/DesignPattern/img/abstract_factory_01.png?raw=true)

### 구현 예제 1

* [code repo](https://github.com/namjunemy/design-pattern/tree/master/08_abstract_factory/src/ex01)

* abst

  * BikeFactory

  * Body
  * Wheel

* alton

  * AltonFactory
  * AltonBody
  * AltonWheel

* sam

  * SamFactory
  * SamBody
  * SamWheel

### 구현 예제 2

* [code repo](https://github.com/namjunemy/design-pattern/tree/master/08_abstract_factory/src/ex02)
  * abst
    * Button
    * TextArea
    * GuiFactory
  * concrete
    * FactoryInstance
  * mac
    * MacButton
    * MacTextArea
    * MacGuiFactory
  * linux
    * LinuxButton
    * LinuxTextArea
    * LinuxGuiFactory
  * ...