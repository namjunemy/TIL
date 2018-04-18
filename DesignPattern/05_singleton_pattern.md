# 5. Singleton Pattern

> 출처 : Head First Design Patterns & 인프런 - '자바 디자인 패턴의 이해'(GoF Design Pattern)
>
> [Code Repository](https://github.com/namjunemy/design_pattern)

"하나의 인스턴스만 있도록 하기"

### 객체란?

* 객체 : 속성과 기능을 갖춘 것
* 클래스 : 속성과 기능을 정의한 것
* 인스턴스 : 속성과 기능을 가진 것 중 실제 하는 것

### 학습 목표

* 싱글톤 패턴을 통해서 하나의 인스턴스만 생성하도록 구현 할 수 있다.
* 싱글톤 패턴 - 하나만 생성해야할 객체를 위한 패턴

### 기본 설계

* 싱글톤 클래스에서 인스턴스로 싱글톤을 하나 가지고 있고,
* 그 인스턴스를 return 하는 메소드를 하나 가지고 있다.

![](https://github.com/namjunemy/TIL/blob/master/DesignPattern/img/singleton_01.png?raw=true)

### 요구 사항

* 개발 중인 시스템에서 싱글톤 스피커 클래스를 만들어 주세요.
* 스피커 클래스가 여러개 생성 가능하다면, 볼륨을 높일 경우 스피커를 사용하는 클래스를 찾아 다니면서 일일히 볼륨을 높여야 하는 경우가 발생한다.

### 구현

```java
public class SystemSpeaker {
  static private SystemSpeaker instance;
  private int volume;

  private SystemSpeaker() {
    volume = 5;
  }

  public static SystemSpeaker getInstance() {
    if (instance == null)
      instance = new SystemSpeaker();
    return instance;
  }

  public int getVolume() {
    return volume;
  }

  public void setVolume(int volume) {
    this.volume = volume;
  }
}
```

```java
public class Main {
  public static void main(String[] args) {
    SystemSpeaker speaker1 = SystemSpeaker.getInstance();
    SystemSpeaker speaker2 = SystemSpeaker.getInstance();

    System.out.println(speaker2.getVolume());
    System.out.println(speaker1.getVolume());

    speaker2.setVolume(11);
    System.out.println(speaker2.getVolume());
    System.out.println(speaker1.getVolume());
  }
}
```