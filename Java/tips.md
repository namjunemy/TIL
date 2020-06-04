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

## 일급컬렉션

* https://jojoldu.tistory.com/412
  * **비지니스에 종속적인 자료구조**가 만들어져, 이후 발생할 문제가 최소화 됨
  * 일급 컬렉션은 **컬렉션의 불변을 보장**
    * java의 final은 재할당만 금지함. 따라서 list.add() 또는 map.put() 등이 가능함.
    * 일급 컬렉션 또는 래퍼 클래스로 만들어서 생성자와 get만 열어 둔다면, 불변을 보장할 수 있음.
  * **값과 로직이 함께 존재**
    * 해당 컬렉션과 그 컬렉션으로 하는 계산 로직은 서로 관계가 있는데,
    * 리스트에 데이터를 담고, 서비스 혹은 util 메서드에서 리스트 데이터를 가져와서 수행함.
    * 이렇게 되면 같은 리스트를 쓰는 여러 개발자에 의해 같은 기능을 하는 메서드를 중복 생성할 수 있는 여지가 생김.
    * 코드가 흩어질 확률이 높음.
    * 따라서, 일급 컬렉션 안에 리스트와 계산식을 같이 두면, 상태와 로직이 한곳에서 관리 됨.
  * 일급컬렉션의 이름으로 명확한 식별에 도움(글 참조)

## Kafka

* Spring-kafka의 KafkaTemplate을 통해 카프카 메세지 전송 하면 ListenableFuture를 반환하는데,

* 퓨처에 콜백 리스너를 등록할 수 있다.

* 전송 후 성공/실패에 따른 동작을 구현할 수 있다.

  ```java
  Message<T> message = MessageBuilder
    .withPayload(data)
    .setHeader(KafkaHeaders.TOPIC, topic)
    .build();
  
  ListenableFuture<SendResult<String, CustomMessage>> future = kafkaTemplate.send(message);
  
  future.addCallback(new ListenableFutureCallback<SendResult<String, CustomMessage>>() {
    @Override
    public void onFailure(Throwable ex) {
      log.debug("fail");
    }
  
    @Override
    public void onSuccess(SendResult<String, CustomMessage> result) {
      log.debug("success");
    }
  });
  
  ```

## Spring

* 팩토리메서드 패턴을 이용해서 스프링에서 빈 동적으로 처리하기
  * 스프링에서는 Bean을 컬렉션으로 주입할 수 있다. 하나의 인터페이스를 상속받고 있는 구현체인 Bean들을 List로 주입받을 수 있다.
  * 이렇게 되면 같은 행동을 하는 다른 타입의 빈들을 모조리 주입받아서 if-else 또는 switch처리하지 않아도 된다.
  * 빈의 의존객체들이 줄어들고, 분기 로직도 없앨 수 있다.
  * 이 방식으로 SQS가 여러 타입의 메세지를 처리해야 하는 상황을 효율적으로 처리할 수 있었다.
    * 이 과정 블로그 포스팅해보자!
  * https://zorba91.tistory.com/306