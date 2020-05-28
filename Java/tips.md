# Tips

> 개발하면서 마주한 팁들

## Enum

* [Java enum의 사용 - 이종립님](https://johngrib.github.io/wiki/java-enum/)
* [이펙티브 자바 38 - 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라](https://jaehun2841.github.io/2019/02/04/effective-java-item38/)

* Enum을 확장해서 사용하려면 interface를 상속 받아서 구현할 수 있다.

* Enum 클래스가 내부에서 기본적으로 Enum\<T\> 를 상속받고 있기 때문에 다른 클래스를 extend 할 수 없다.

* Enum 상수 하나당 인스턴스가 만들어지며, 각각 public static final 이다.

* ENUM 타입별로 다르게 동작하는 코드를 구현하고 싶다면, 아래와 같은 방식이 있다.

  ```java
  public enum Operation {
      PLUS {
          @Override
          public double apply(double x, double y) { return x + y; }
      },
      MINUS {
        	@Override
          public double apply(double x, double y) { return x - y; }
      },
      TIMES {
        	@Override
          public double apply(double x, double y) { return x * y; }
      },
      DIVIDE {
        	@Override
          public double apply(double x, double y) { return x / y; }
      };
    
      public abstract double apply(double x, double y);
  }
  ```

* 예를 들어, VOD 또는 IMG가 저장될 수 있는 Storage 엔티티에서 FileType에 따라서 다른 행동을 구현하고 싶었을 때 위와 같이 해결 할 수 있다.

