# 4. Factory Method Pattern

> 출처 : Head First Design Patterns & 인프런 - '자바 디자인 패턴의 이해'(GoF Design Pattern)
>
> [Code Repository](https://github.com/namjunemy/design_pattern)

"팩토리 메소드 패턴에서는 객체를 생성하기 위한 인터페이스를 정의하는데, 어떤 클래스의 인스턴스를 만들지는 서브클래스에서 결정하게 만든다. 팩토리 메소드 패턴을 이용하면 클래스의 인스턴스를 만드는 일을 서브클래스에게 맡긴다."

### 학습목표

* 팩토리 메소드 패턴에서 '**템플릿 메소드 패턴이 사용된다는 것**'을 안다.
* 팩토리 메소드 패턴에서 **구조와 구현의 분리**를 이해하고 그것의 장점을 이해한다.

### 기본 설계

* 생산자(Creator) 클래스와 제품(Product) 클래스의 구조가 존재한다.
* Creator에는 제품을 가지고 원하는 일을 하기 위한 모든 메소드들이 구현되어 있다. 하지만, 제품을 만들어 주는 팩토리 메소드는 추상 메소드로 정의되어 있을 뿐 구현되어 있진 않다. Creator의 모든 서브클래스에서 abstract 메소드를 구현해야 한다.
  * 실제로 제품을 만드는 create()메소드(이것이 팩토리 메소드이다.)는 템플릿 메소드 패턴으로 구현되어 제품이 만들어 지는 과정에 대한 abstract 메소드들을 포함한다.
* ConcreteCreator에서는 실제로 제품을 생산하는 abstract 메소드들을 구현한다.
* Product에서는 모두 똑같은 인터페이스를 구현해야 한다. 그래야 그 제품을 사용할 클래스에서 구상클래스가 아닌 인터페이스에 대한 레퍼런스를 써서 객체를 참조할 수 있다.
* 구상 클래스(<=>추상 클래스) 인스턴스를 만들어내는 일은 ConcreteCreator가 책임진다. 여기에서 실제 제품을 만들어내는 방법을 알고 있는 클래스는 이 클래스 뿐이다.

![](https://github.com/namjunemy/TIL/blob/master/DesignPattern/img/factory_method_01.png?raw=true)

### 요구 사항

* 게임 아이템과 아이템 생성을 구현해주세요.
  * 아이템을 생성하기 전에 데이터베이스에서 아이템 정보를 요청합니다.
  * 아이템을 생성 후 아이템 복제 등의 불법을 방지하기 위해 데이터 베이스에 아이템 생성 정보를 남깁니다
* 아이템을 생성하는 주체를 ItemCreator로 이름 짓습니다.
* 아이템은 item이라는 interface로 다룰 수 있도록 합니다.
  * item은 use 함수를 기본 함수로 갖고 있습니다.
* 현재 아이템의 종류는 체력 회복 물약, 마력 회복 물약이 있습니다.

  

### 구현

* [code repo](https://github.com/namjunemy/design-pattern/tree/master/04_factory_method_pattern/src)


* 디렉토리 구조
* factory_method
  * Framework
    - Item(Interface)
    - ItemCreator
  * concrete
    * HpPotion
    * HpCreator
    * MpPorion
    * MpCreator
    * Main

```java
package framework;

public interface Item {
  public void use();
}
```

```java
package framework;

public abstract class ItemCreator {
  public Item create() {
    requestItemsInfo();
    Item item = createItem();
    createItemLog();

    return item;
  }

  abstract protected void requestItemsInfo();
  abstract protected void createItemLog();
  abstract protected Item createItem();
}
```

  

```java
package concrete;

import framework.Item;
import framework.ItemCreator;

import java.util.Date;

public class HpCreator extends ItemCreator{
  @Override
  protected void requestItemsInfo() {
    System.out.println("데이터베이스에서 체력 회복 물약의 정보를 가져온다.");
  }

  @Override
  protected void createItemLog() {
    System.out.println("체력 회복 물약을 새로 생성 했습니다." + new Date());
  }

  @Override
  protected Item createItem() {
    return new HpPotion();
  }
}
```

```java
package concrete;

import framework.Item;

public class HpPotion implements Item {
  @Override
  public void use() {
    System.out.println("체력 회복");
  }
}
```

  

```java
package concrete;

import framework.Item;
import framework.ItemCreator;

public class Main {
  public static void main(String[] args) {
    ItemCreator itemCreator;
    Item item;

    itemCreator = new HpCreator();
    item = itemCreator.create();
    item.use();

    itemCreator = new MpCreator();
    item = itemCreator.create();
    item.use();
  }
}
```

```shell
데이터베이스에서 체력 회복 물약의 정보를 가져온다.
체력 회복 물약을 새로 생성 했습니다.Tue Apr 17 02:15:08 KST 2018
체력 회복
데이터베이스에서 마력 회복 물약의 정보를 가져온다.
마력 회복 물약을 새로 생성 했습니다.Tue Apr 17 02:15:08 KST 2018
마력 회복

Process finished with exit code 0
```