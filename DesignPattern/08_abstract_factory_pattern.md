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

### 추상 팩토리 패턴

추상 팩토리 패턴에서는 인터페이스를 이용하여 서로 연관된, 또는 의존하는 객체를 구상 클래스를 지정하지 않고도 생성할 수 있다.

추상 팩토리 패턴을 사용하면 클라이언트에서 추상 인터페이스를 통해서 일련의 제품들을 공급받을 수 있다. 이때, 실제로 어떤 제품이 생상되는지는 전혀 알 필요도 없다. 따라서 클라이언트와 팩토리에서 생산되는 제품을 분리시킬 수 있다. 

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