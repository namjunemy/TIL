# 멀티 스레드

> '이것이 자바다 - 신용권' 12장 학습
>
> [소스코드 repo](https://github.com/namjunemy/this_is_java)
>
> 1절. 멀티 스레드 개념
>
> 2절. 작업 스레드 생성과 실행
>
> 3절. 스레드 우선순위
>
> 4절. 동기화 메소드와 동기화 블록
>
> 5절. 스레드 상태
>
> 6절. 스레드 상태 제어
>
> 7절. 데몬 스레드
>
> 8절. 스레드 그룹
>
> 9절. 스레드 풀



## 1. 프로세스와 스레드

### 프로세스

* 실행 중인 하나의 프로그램
  * 하나의 프로그램은 다중 프로세스를 만들기도 한다.

### 멀티 태스킹

* 두 가지 이상의 작업을 동시에 처리하는 것
* 멀티 프로세스 : 독립적으로 프로그램들을 실행하고 여러 가지 작업 처리
* 멀티 스레드 : 한 개의 프로그램을 실행하고 내부적으로 여러 가지 작업 처리

### 메인 스레드

* 모든 자바 프로그램은 메인 스레드가 main() 메소드를 실행하면서 시작된다.
* main() 메소드의 첫 코드부터 아래로 순차적으로 실행한다.
* main() 메소드의 마지막 코드를 실행하거나, return 문을 만나면 실행이 종료된다.
* main 스레드(JVM이 생성)는 작업 스레드를 만들어서 병렬로 코드를 실행할 수 있다. 즉 멀티 스레드를 생성해서 멀티 태스킹을 수행한다.
* 프로세스의 종료
  * 싱글 스레드 : 메인 스레드가 종료하면 프로세스도 종료된다.
  * 멀티 스레드 : 실행 중인 스레드가 하나라도 있다면, 프로세스는 종료되지 않는다. 




## 2. 작업 스레드 생성과 실행

> 몇 개의 작업을 병렬로 실행할지 결정해야 한다.
>
> 작업 스레드 생성 방법
>
> * Thread 클래스로부터 직접 생성
> * Thread 하위 클래스로부터 생성



### Thread 클래스로부터 직접 생성

java.lang.Thread 클래스로부터 작업 스레드 객체를 직접 생성 하려면 Runnable을 매개값으로 갖는 생성자를 호출해야 한다. Runnable은 작업 스레드가 실행할 수 있는 코드를 가지고 있는 객체라고 해서 붙여진 이름이다. Runnable에는 run() 메소드 하나가 정의되어 있는데, 구현 클래스는 run()을 재정의해서 작업 스레드가 실행할 코드를 작성해야 한다.

~~~java
class Task implements Runnable {
  public void run() {
    스레드가 실행할 코드;
  }
}
~~~

* 첫번째 방법 - Runnable 구현 객체를 생성한 후, 이것을 매개값으로 Thread 생성자를 호출

  ~~~java
  Runnable task = new Task();

  Thread thread = new Thread(task);
  ~~~

* 두번째 방법 - 사실상 첫번째 방법과 동일하지만, Thread 생성자를 호출할 때 Runnable 익명 객체를 매개값으로 사용할 수 있다. 이 방법이 더 많이 사용된다. 

  ~~~java
  Thread thread = new Thread( new Runnable() {
    public void run() {
      스레드가 실행할 코드;
    }
  });
  ~~~

* 세번째 방법 - 람다식을 매개값으로 사용하는 방법

  Runnable 인터페이스는 run() 메소드 하나만 정의되어 있기 때문에 함수적 인터페이스이다. 따라서 람다식을 매개값으로 사용할 수도 있따. 가장 간단하지만, 자바 8부터 지원된다.

  ~~~java
  Thread thread = new Thread( () -> {
      스레드가 실행할 코드;
  });
  ~~~

이렇게 생성된 스레드가 start() 메소드를 실행할 때, run() 메소드가 실행 된다.



### Thread 하위 클래스로부터 생성

작업 스레드가 실행할 작업을 Runnable로 만들지 않고, Thread의 하위 클래스로 작업 스레드를 정의하면서 작업 내용을 포함시킬 수도 있다. Thread 클래스를 상속한 후 run 메소드를 overriding해서 스레드가 실행할 코드를 작성하면 된다.

~~~java
public class WorkerThread extends Thread {
  @Override
  public void run() {
      //스레드가 실행할 코드
  }
}
~~~

~~~java
Thread thread = new Thread() {
  public void run() {
      //스레드가 실행할 코드
  }
}
~~~

~~~java
thread.start()
~~~



### 스레드의 이름

* 메인 스레드 이름 : main

* 작업 스레드 이름 : Thread-n

  ~~~java
  thread.getName();
  ~~~

* 작업 스레드의 이름 변경

  ~~~java
  thread.setName("스레드 이름")
  ~~~

* 코드를 실행하는 스레드의 참조 얻기

  ~~~java
  Thread thread = Thread.currentThread();
  ~~~




## 3. 스레드 우선 순위

### 동시성과 병렬성

* 동시성(Concurrency)

  * 멀티 작업을 위해 하나의 코어에서 멀티 스레드가 번갈아 가며 실행하는 성질

* 병렬성(Parallelism)

  * 멀티 작업을 위해 멀티 코어에서 개별 스레드를 동시에 실행하는 성질

    ​

멀티 스레드는 동시성 또는 병렬성으로 실행되기 때문에 이 용어들에 대해서 정확히 이해하는 것이 좋다. 동시성은 멀티 작업을 위해 하나의 코어에서 멀티 스레드가 번갈아가며 실행하는 설질을 말하고, 병렬성은 멀티 작업을 위해 멀티 코어에서 개별 스레드를 동시에 실행하는 성질을 말한다. 싱글 코어 CPU를 이용한 멀티 스레드 작업은 병렬적으로 실행되는 것처럼 보이지만, 사실은 번갈아가며 실행하는 동시성 작업이다. 번갈아 실행하는 것이 워낙 빨라서 병렬성으로 보일 뿐이다.



### 스레드 스케줄링

* 스레드 의 개수가 코어의 수보다 많을 경우
  * 스레드를 어떤 순서에 의해 동시성으로 실행할 것인가를 결정해야 하는데, 이것을 스레드 스케줄링이라고 한다.
  * 스레드 스케줄링에 의해 스레드들은 아주 짧은 시간에 번갈아가면서 그들의 run() 메소드를 조금씩 실행한다.
* 자바의 스레드 스케쥴링
  * 우선 순위(Priority) 방식과 순환 할당(Round-Robin) 방식을 사용
    * 우선 순위 방식(코드로 제어 가능) : 우선 순위가 높은 스레드가 실행 상태를 더 많이 가지도록 스케쥴링
    * 순환 할당 방식(코드로 제어할 수 없음) : 시간 할당량(Time Slice)을 정해서 하나의 스레드를 정해진 시간만큼 실행하는 방식



### 스레드 우선 순위

* 스레드들이 동시성을 가질 경우 우선적으로 실행할 수 있는 순위

* 우선 순위는 1(낮음)에서부터 10(높음)까지로 부여

  * 모든 스레드들은 기본적으로 5의 우선 순위를 할당

* 우선 순위 변경 방법

  ~~~java
  thread.setPriority(우선순위);

  thread.setPriority(Thread.MAX_PRIORITY);
  thread.setPriority(Thread.NORM_PRIORITY);
  thread.setPriority(Thread.MIN_PRIORITY);
  ~~~

* 우선 순위 효과

  * 싱글 코어 경우
    * 우선 순위가 높은 스레드가 실행 기회를 더 많이 가지기 때문에
    * 우선 순위가 낮은 스레드 보다 계산 작업을 빨리 끝낸다.
  * 멀티 코어 경우
    * 쿼드의 경우에는 4개의 스레드가 병렬성으로 실행될 수 있기 때문에
    * 4개 이하의 스레드를 실행할 경우에는 우선 순위 방식은 크게 영향을 미치지 못한다.
    * 최소한 5개 이상의 스레드가 실행되어야 우선 순위의 영향을 받는다.




## 4. 동기화 메소드와 동기화 블록

### 공유 객체를 사용할 때의 주의할 점

싱글 스레드 프로그램에서는 한 개의 스레드가 객체를 독차지해서 사용하면 되지만, 멀티 스레드 프로그램에서는 스레드들이 객체를 공유해서 작업해야 하는 경우가 있다. 이 경우, 스레드 A를 사용하던 객체가 스레드 B에 의해 상태가 변경될 수 있기 때문에 스레드 A가 의도했던 것과는 다른 결과를 산출할 수도 있다.

* 멀티 스레드가 하나의 객체를 공유함에따라 생기는 오류

  ​

### 동기화 메소드 및 동기화 블록 - Synchronized

멀티 스레드 프로그램에서 단 하나의 스레드만 실행할 수 있는 코드 영역을 임계 영역(critical section)이라고 한다. 자바는 임계 영역을 저장하기 위해 동기화(synchronized) 메소드와 동기화 블록을 제공한다. 스레드가 객체 내부의 동기화 메소드 또는 블록에 들어가면 즉시 객체에 잠금을 걸어 다른 스레드가 임계 영역 코드를 실행하지 못하도록 한다. 동기화 메소드를 만드는 방법은 메소드 선언에 synchronized 키워드를 붙이면 된다. synchronized 키워드는 인스턴스와 정적 메소드 어디든 붙일 수 있다. 

* 단 하나의 스레드만 실행할 수 있는 메소드 또는 블록을 말한다.

* 다른 스레드는 메소드나 블록의 실행이 끝날 때까지 대기해야 한다.

* 동기화 메소드

  동기화 메소드는 메소드 전체 내용이 임계 영역이므로 스레드가 동기화 메소드를 실행하는 즉시 객체레는 잠금이 일어나고, 스레드가 동기화 메소드를 실행 종료하면 잠금이 풀린다. 메소드 전체 내용이 아니라, 일부 내용만 임계 영역으로 만들고 싶다면 아래와 같이 동기화 블록을 만들면 된다.

  ```java
  public synchronized void method() {
    임계 영역; //단 하나의 스레드만 실행
  }
  ```

* 동기화 블록

  동기화 블록의 외부 코드들은 여러 스레드가 동시에 실행할 수 있지만, 동기화 블록의 내부 코드는 임계 영역이므로 한 번에 한 스레드만 실행할 수 있고 다른 스레드는 실행할 수 없다. 만약 동기화 메소드와 동기화 블록이 여러 개 있을 경우, 스레드가 이들 중 하나를 실행할 때 다른 스레드는 해당 메소드는 물론이고 다른 동기화 메소드 및 블록도 실행할 수 없다. 하지만 일반 메소드는 실행이 가능하다.

  ```java
  public vod method () {
    //여러 스레드가 실행 가능한 영역
    ...
      
    synchronized(공유객체) {
      임계 영역; //단 하나의 스레드만 실행
    }
    
    //여러 스레드가 실행 가능한 영역
    ...
  }
  ```




## 5. 스레드 상태

스레드 객체를 생성하고, start() 메소드를 호출하면 곧바로 스레드가 실행되는 것처럼 보이지만 사실은 실행대기 상태가 된다. 실행 대기 상태란 아직 스케줄링 되지 않으서 실행을 기다리고 있는 상태를 말한다. 실행 대기 상태에 있는 스레드 중에서 스레드 스케줄링으로 선택된 스레드가 비로소 CPU를 점유하고 run() 메소드를 실행한다. 이 상태를 '실행(Running) 상태'라고 한다. 실행 상태의 스레드는 run() 메소드를 모두 실행하기 전에 스레드 스케줄링에 의해 다시 실행 대기 상태로 돌아 갈 수 있다. 그리고 실행 대기 상태에 있는 다른 스레드가 선택되어 실행 상태가 된다. 이렇게 스레드는 실행 대기 상태와 실행 상태를 번갈아가면서 자신의 run() 메소드를 조금씩 실행한다. 실행 상태에서 run() 메소드가 종료되면, 더 이상 실행할 코드가 없기 때문에 스레드의 실행은 멈추게 된다. 이 상태를 '종료 상태' 라고 한다.

![img_1](https://github.com/namjunemy/TIL/blob/master/Java/img/12_1.jpg?raw=true)



경우에 따라서 스레드는 실행 상태에서 실행대기 상태로 가지 않을 수도 있다. 실행 상태에서 일시 정지 상태로 가기도 하는데, 일시 정지 상태는 스레드가 실행할 수 없는 상태이다. 일시 정지 상태는 WAITING, TIMED_WAITING, BLOCKED가 있는데, 일시 정지 상태가 되는 이유는 나중에 설명하도록 하고, 스레드가 다시 실행 상태로 가기 위해서는 일시 정지 상태에서 실행 대기 상태로 가야한다.

이러한 스레드의 상태를 코드에서 확인할 수 있도록 하기위해 자바 5부터 Thread 클래스에 getState() 메소드가 추가되었다. getState() 메소드는 다음 표처럼 스래드 상태에 따라서 Thread.State 열거 상수를 리턴한다.

```java
Thread targetThread = new TargetThread();

Thread.State state = targetThread.getState();
```

|     열거 상수     |  상태   |                   설명                   |
| :-----------: | :---: | :------------------------------------: |
|      NEW      | 객체 생성 | 스레드 객체가 생성, 아직 start() 메소드가 호출되지 않은 상태 |
|   RUNNABLE    | 실행 대기 |         실행 상태로 언제든지 갈 수 있는 상태          |
|    WAITING    | 일시정지  |        다른 스레드가 통지할 때까지 기다리는 상태         |
| TIMED_WAITING |       |           주어진 시간 동안 기다리는 상태            |
|    BLOCKED    |       |     사용하고자 하는 객체의 락이 풀릴 때까지 기다리는 상태     |
|  TERMINATED   |  종료   |               실행을 마친 상태                |



## 6. 스레드 상태 제어

사용자는 미디어 플레이어에서 동영상을 보다가 일시 정지시킬 수도 있고, 종료시킬 수도 있다. 일시 정지는 조금 후 다시 동영상을 보겠다는 의미이므로 미디어 플레이어는 동영상 스레드를 일시 정지 상태로 만들어야 한다. 그리고 종료는 더 이상 동영상을 보지 않겠다는 의미 이므로 미디어 플레이어는 스레드를 종료 상태로 만들어야 한다. 이와 같이 실행 중인 스레드의 상태를 변경하는 것을 스레드 상태 제어라고 한다.

멀티 스레드 프로그램을 만들기 위해서는 정교한 스레드 상태 제어가 필요한데, 상태 제어가 잘못되면 프로그램은 불안정해지고 다운된다. 멀티 스레드 프로그래밍이 어려운 이유가 여기에 있다.스레드를 잘 사용하면 약이 되지만, 잘못 사용하면 치명적인 프로그램 버그가 되기 때문에 스레드를 정확하게 제어하는 방법을 잘 알고 있어야 한다.

스레드 제어를 제대로 하기 위해서는 스레드의 상태 변화를 가져오는 메소드들을 파악하고 있어야 한다. 다음 그림은 상태 변화를 가져오는 메소드의 종류를 보여준다.

![img_2](https://github.com/namjunemy/TIL/blob/master/Java/img/12_2.jpg?raw=true)

처음에 스레드를 생성하게 되면 실행 대기 상태가 된다. 실행 대기 상태에서 CPU 스케줄러에 의해 선택이되면 실행이 된다. 실행 상태에서 시간 할당량이 다 되면 실행 대기 상태로 가는데, 시간 할당량이 끝나기 전에 **yield()** 메소드를 호출 하게 되면 실행 대기로 만들 수도 있다. 

스레드가 실행중일 때,

* sleep() 메소드를 호출하게 되면 주어진 시간동안 일시 정지 상태가 되고, 시간이 지나면 실행 대기가 된다.
* join() 메소드를 호출하게 되면, join() 메소드를 호출 한 스레드가 종료 될 때 까지 일시 정지 되었다가 실행 대기 상태로 이동한다.
* wait() 메소드를 호출하게 되면 역시 일시 정지 상태가 된다. 하지만 wait() 메소드로 일시 정지 상태에 들어간 스레드는 자기 혼자서 실행 대기 상태로 갈 수 없다. 따라서, 다른 스레드가 notify(), notifyAll()을 실행해야 실행 대기로 갈 수 있다.

일시 정지 상태에 있는 스레드에서

* interrupt() 메소드가 호출 되면, Exception이 발생함과 동시에 일시 정지 상태에서 풀리면서 실행 대기 상태로 갈 수 있다.

실행 중인 스레드에서

* stop() 메소드를 호출하면 스레드를 종료시킬 수 있다. 하지만, Deprecated 되었다. 이유는 [블로그 포스팅](http://ict-nroo.tistory.com/22)을 참고하자.



### sleep() - 주어진 시간 동안 일시정지

~~~java
try {
  Thread.sleep(1000);
} catch(InterruptedException e) {
  //interrupt() 메소드가 호출되면 실행
}
~~~

* 얼마 동안 일시 정지 상태로 있을 것인지, 밀리세컨드(1/1000) 단위로 지정
* 일시 정지 상태에서 interrupt() 메소드가 호출되면 InterruptedException이 발생함. 따라서 try-catch로 묶여 있어야 함.



### yield() - 다른 스레드에게 실행 양보

실행 대기 상태에 있는 스레드A과 스레드B가 있다고 가정하자. 스레드A이 실행 중일 때, yeild() 메소드를 호출하면, 스레드A는 실행 대기 상태가 되고 동일 또는 높은 우선 순위를 가진 스레드B에게 실행을 양보한다.

~~~java
##YieldExample.java

public class YieldExample {
  public static void main(String[] args) {
    ThreadA threadA = new ThreadA();
    ThreadB threadB = new ThreadB();

    threadA.start();
    threadB.start();

    try {
      Thread.sleep(3000);
    } catch (Exception e) {
    }
    threadA.work = false;

    try {
      Thread.sleep(3000);
    } catch (Exception e) {
    }
    threadA.work = true;

    try {
      Thread.sleep(3000);
    } catch (Exception e) {
    }
    threadA.stop = true;
    threadB.stop = true;

  }
}

##ThreadA.java
public class ThreadA extends Thread {
  public boolean stop = false;
  public boolean work = true;

  public void run() {
    while(!stop) {
      if(work) {
        System.out.println("Thread A 작업중");
      } else {
        Thread.yield();
      }
    }
    System.out.println("Thread A 종료");
  }
}

##ThreadB.java
public class ThreadB extends Thread {
  public boolean stop = false;
  public boolean work = true;

  public void run() {
    while(!stop) {
      if(work) {
        System.out.println("Thread B 작업중");
      } else {
        Thread.yield();
      }
    }
    System.out.println("Thread B 종료");
  }
}

~~~



### join() - 다른 스레드의 종료를 기다림

ThreadA와 ThreadB가 있다고 가정하자.

**ThreadA**

~~~java
threadB.start();
threadB.join();
~~~

**ThreadB**

~~~java
run() {
    
}
~~~

ThreadA에서 threadB의 start()를 호출하고, threadB.join() 메소드를 호출하면 ThreadA는 threadB의 run() 메소드의 실행이 종료 될 때 까지 일시 정지 상태가 된다. ThreadB의 run() 메소드가 종료되면 비로소 ThreadA는 다음 코드를 실행하게 된다. 헷갈리지 말자 ThreadA가 일시 정지 상태가 된다! join() 을 호출하는 쪽에서 조인하길 기다리는 것이다!

보통 ThreadB의 연산 결과가 필요한 ThreadA 스레드에서 많이 사용 한다.



### 스레드 간 협업 - wait(), notify(), notifyAll()

이 메소드들은 스레드가 가지고 있는 메소드가 아니다. sleep(), yield(), join()은 스레드가 가지고 있는 메소드이지만, 위의 메소드들은 Object가 가지고 있는 메소드이다. 이 말은, 모든 객체가 가지고 있는 메소드라는 이야기다.

* **동기화 메소드 또는 동기화 블록에서만 호출 가능한 Object 메소드이다.**

* 두 개의 스레드가 교대로 번갈아 가며 실행해야 할 경우에 주로 사용한다.(공유 객체 - ex)생산자 소비자 스레드)

  * 생산자 스레드가 공유 객체에 데이터를 저장하고, 소비자 스레드에서 데이터를 읽는 작업을 수행할 때를 생각해보자. 생산자 스레드에서 공유객체에 저장한 데이터를 소비자 스레드에서 두번 중복해서 처리하면 비 효율적인 처리 작업이 된다. 따라서 생산자 스레드에서 data필드에 값을 넣고 notify()를 호출해서 소비자 스레드가 data를 읽게 하고, data를 읽은 소비자 스레드는 다시 공유 객체를 비우고 notify()를 호출해서 생산자 스레드가 data필드에 값을 넣을 수 있도록 해준다. 코드로 보는 것이 이해가 빠르겠다.. **다시 한번 강조하지만 wait(), notify(), notifyAll()은 동기화 메소드 또는 동기화 블록에서만 호출 가능한 Object 메소드이다.**

    ~~~java
    ## DataBox.java

    public class DataBox {
      private String data;

      public synchronized String getData() {
        if (this.data == null) {
          try {
            wait();
          } catch (InterruptedException e) {
          }
        }
        String returnValue = data;
        System.out.println("ConsummerThread가 읽은 데이터 : " + returnValue);
        data = null;
        notify();
        return returnValue;
      }

      public synchronized void setData(String data) {
        if (this.data != null) {
          try {
            wait();
          } catch (InterruptedException e) {

          }
        }
        this.data = data;
        System.out.println("ProducerThread가 생성한 데이터 : " + data);
        notify();
      }
    }
    ~~~

    ~~~java
    ##ConsumerThread.java

    public class ConsumerThread extends Thread {
      private DataBox dataBox;

      public ConsumerThread(DataBox dataBox) {
        this.dataBox = dataBox;
      }

      @Override
      public void run() {
        for (int i = 1; i <= 3; i++) {
          String data = dataBox.getData();
        }
      }
    }
    ~~~

    ~~~java
    ##ProducerThread.java

    public class ProducerThread extends Thread {
      private DataBox dataBox;

      public ProducerThread(DataBox dataBox) {
        this.dataBox = dataBox;
      }

      @Override
      public void run() {
        for (int i = 1; i <= 3; i++) {
          String data = "Data=" + i;
          dataBox.setData(data);
        }
      }
    }
    ~~~


* wait()
  * 호출한 스레드는 일시 정지 상태가 되어서, waiting pool에서 관리가 된다.
  * wait()로 일시정지 상태가 된 스레드는 절대 자기 혼자서 실행 대기 상태로 갈 수 없다.
  * 실행 상태에 있는 다른 스레드가, notify() 또는 notifyAll() 메소드를 호출 해줘야 실행 대기 상태로 갈 수 있다.
* wait(long timeout), wait(long timeout, int nanos)
  * wait() 메소드가 오버로딩 된 메소드로 timeout 만큼의 시간이 지난 후 실행 대기 상태로 갈 수 있고,
  * 이 메소드들 역시 notify() 또는 notifyAll() 메소드를 호출 해주면 실행 대기 상태로 갈 수 있다.



### 스레드의 안전한 종료 - stop 플래그, interrupt()

해당 내용은 [블로그 포스팅](http://ict-nroo.tistory.com/22)을 참고하자.



## 7. 데몬 스레드

* 주 스레드의 작업을 돕는 보조적인 역할을 수행하는 스레드

* 주 스레드가 종료되면 데몬 스레드는 강제적으로 자동 종료

  * 워드 프로세서의 자동저장, 미디어플레이어의 동영상 및 음악 재생, 가비지 컬렉터 등

* 데몬 스레드 설정

  * 주 스레드가 데몬이 될 스레드의 setDaemon(true)를 호출

    ~~~java
    public static void main(String[] args) {
      AutoSaveThread thread = new AutoSaveThread();
      thread.setDaemon(true);
      thread.start();
      
      ...
    }
    ~~~

  * 반드시 start() 메소드 호출 전에 setDaemon(true)를 호출해야 한다.

    * 그렇지 않으면 IllegalThreadStateException이 발생한다.

* 데몬 레드 확인 방법

  * isDamon() 메소드의 리턴값을 조사(true면 데몬스레드, false면 주 스레드)





## 8. 스레드 그룹

스레드 그룹은 관련된 스레드를 묶어서 관리할 목적으로 이용된다. JVM이 실행되면 system 스레드 그룹을 만들고, JVM 운영에 필요한 스레드들을 생성해서 system 스레드 그룹에 포함시킨다. 그리고 system의 하위 스레드 그룹으로 main을 만들고 메인 스레드를 main 스레드 그룹에 포함시킨다. 스레드는 반드시 하나의 스레드 그룹에 포함되는데, 명시적으로 스레드 그룹에 포함시키지 않으면 기본적으로 자신을 생성한 스레드와 같은 스레드 그룹에 속하게 된다. 우리가 생성하는 작업 스레드는 대부분 main스레드가 생성하므로 기본적으로 main 스레드 그룹에 속하게 된다. 



### 스레드 그룹 이름 얻기

~~~java
ThreadGroup group = Thread.currentThread.getThreadGroup();

String groupName = group.getName();
~~~



### 스레드 그룹 생성

~~~java
ThreadGroup tg = new ThreadGroup(String name);
ThreadGroup tg = new ThreadGroup(ThreadGroup parent, String name);
~~~

* 부모 그룹을 지정하지 않으면 현재 스레드가 속한 그룹의 하위 그룹으로 생성

* 스레드를 그룹에 명시적으로 포함시키는 방법

  ~~~java
  //Runnable 구현상속
  Threat t = new Thread(ThreadGroup group, Runnable target);
  Threat t = new Thread(ThreadGroup group, Runnable target, String name);
  Threat t = new Thread(ThreadGroup group, Runnable target, String name, long stackSize);

  //Thread 상속
  Threat t = new Thread(ThreadGroup group, Stringtarget);
  ~~~



### 스레드 그룹의 일괄 interrupt()

스레드를 스레드 그룹에 포함시키면 어떤 이점이 있을까?

스레드 그룹에서 제공하는 interrupt() 메소드를 이용하면 그룹 내에 포함된 모든 스레드들을 일괄 interrupt할 수 있다. 예를 들어 10개의 스레드들을 모두 종료시키기 위해 각 스레드에서 interrupt() 메소드를 10번 호출할 수도 있지만, 이 스레드들이 같은 스레드 그룹에 소속되어 있을 경우, 스레드 그룹의 interrupt() 메소드를 한번만 호출해주면 된다. 이것이 가능한 이유는 스레드 그룹의 interrupt() 메소드는 포함된 모든 스레드의 interrupt() 메소드를 내부적으로 호출해주기 때문이다.

중요한 것은, 스레드 그룹의 interrupt() 메소드는 소속된 스레드의 interrupt() 메소드를 호출만 할 뿐 개별 스레드에서 발생하는 InterruptedException에 대한 예외 처리를 하지 않는다. 따라서 안전한 종료를 위해서는 개별 스레드가 예외 처리를 해야 한다.

소스코드는 this_is_java repo의 [ch12_멀티스레드/src/thread_group/interrupt](https://github.com/namjunemy/this_is_java/tree/master/ch12_%EB%A9%80%ED%8B%B0%EC%8A%A4%EB%A0%88%EB%93%9C/src/thread_group/interrupt)를 참조하자.



## 9. 스레드 풀(ExecutorService)

### 스레드 폭증

* 병렬 작업 처리가 많아지면 스레드의 개수가 증가한다.
* 스레드 생성과 스케쥴링으로 인해 CPU가 바빠지고, 메모리 사용량이 늘어난다.
* 따라서 애플리케이션의 성능이 급격히 저하된다.



### 스레드 풀(Thread Pool)

* 작업 처리에 사용되는 스레드를 제한된 개수만큼 미리 생성
* 작업 큐(Queue)에 들어오는 작업들을 하나씩 스레드가 맡아 처리한다.
* 작업 처리가 끝난 스레드는 작업 결과를 애플리케이션으로 전달한다.
* 스레드는 다시 작업큐에서 새로운 작업을 가져와 처리



### ExecutorService 인터페이스와 Executors 클래스

* 스레드풀을 생성하고 사용할 수 있도록 java.util.concurrent 패키지에서 제공
* Executors의 정적 메소드를 이용해서 ExecutorService 구현 객체 생성
* 스레드 풀 = ExecutorService 객체



### 스레드풀 생성

* 다음 두가지 메소드 중 하나로 간편 생성하는 방법

  * ```newCachedThreadPool()```

    * 초기 스레드 수 :  0 / 코어 스레드 수 : 0 / 최대 스레드 수 : Integer.MAX_VALUE

    * Int 값이 가질 수 있는 최대 값 만큼 스레드가 추가되나, 운영체제의 상황에 따라 달라진다.

    * 1개 이상의 스레드가 추가되었을 경우, 60초 동안 추가된 스레드가 아무 작업을 하지 않으면

    * 추가된 스레드를 종료하고 풀에서 제거한다.

      ```java
      ExecutorService executorService = Executors.newCachedThreadPool();
      ```

  * ```newFixedThreadPool(int nThreads)```

    * 초기 스레드 수 :  0 / 코어 스레드 수 : nThreads / 최대 스레드 수 : nThreads

    * 코어 스레드 개수와 최대 스레드 개수가 매개값으로 준 nThread이다.

    * 스레드가 작업을 처리하지 않고 놀고 있더라고 스레드 개수가 줄 지 않는다.

    * 꼭 코어의 수만큼 생성할 필요 없다.

      ```java
      ExecutorService executorService = Executors.newFixedThreadPool(
        Runtime.getRuntime().availableProcessors();	//가능한 코어의 수
      )
      ```

* ThreadPoolExecutor을 이용한 직접 생성

  * newCachedThreadPool()과 newFixedThreadPool(int nThreads)가 내부적으로 생엉

  * 스레드의 수를 자동으로 관리하고 싶을 경우 직접 생성해서 사용한다.

  * Ex)

    * 코어 스레드 개수가 3, 최대 스레드 개수가 100인 스레드 풀을 생성
    * 3개를 제외한 나머지 추가된 스레드가 120초 동안 놀고 있을 경우
    * 해당 스레드를 제거해서 스레드 수를 관리

    ```java
    ExecutorService threadPool = new ThreadPoolExecutor(
      3,  //코어 스레드 개수
      100,  //최대 스레드 개수
      120L,	//놀고 있는 시간
      TimeUnit.SECONDS,	//놀고 있는 시간 단위
      new SynchronousQueue<Runnable>()	//작업큐 객체
    )
    ```



### 스레드풀 종료

* 스레드풀의 스레드는 기본적으로 데몬 스레드가 아니다.
  * Main 스레드가 종료되더라도 스레드풀의 스레드는 작업을 처리하기 위해 계속 실행되므로 애플리케이션은 종료되지 않는다.
  * 따라서 스레드풀을 종료해서 모든 스레드를 종료시켜야 한다.
* 스레드풀 종료 메소드
  * ```void shutdown()```
    * 현재 처리 중인 작업뿐만 아니라 작업큐에 대기하고 있는 모든 작업을 처리한 뒤에 스레드풀을 종료시킨다.
  * ```List<Runnable> shutdownNow()```
    * 현재 작업 처리중인 스레드를 interrupt 해서 작업 중지를 시도하고 스레드풀을 종료시킨다. 리턴값 List\<Runnable\>는 작업큐에 있는 미처리된 작업(Runnable 객체)의 목록이다.
  * ```boolean awaitTermination(long timeout, TimeUnit unit)``` 
    * shutdown() 메소드 호출 이후, 모든 작업 처리를 timeout 시간 내에 완료하면 true를 리턴하고, 완료하지 못하면 작업 처리 중인 스레드를 interrupt 하고 false를 리턴한다.



### 작업 생성

* 하나의 작업은 Runnable 또는 Callable 객체로 표현한다.

* Runnable과 Callable의 차이점

  * 작업 처리 완료 후 리턴값이 있느냐 없느냐이다.

  * Runnable 구현 클래스

    ```java
    Runnable task = new Runnable() {
      @Override
      public void run() {
        //스레드가 처리할 작업 내용
      }
    }
    ```

  * Callable 구현 클래스

    ```java
    Callable<T> task = new Callable<T> {
      @Override
      public T call() throws Exception {
        //스레드가 처리할 작업 내용
        return T;
      }
    }
    ```

* 스레드풀에서의 작업 처리는

  * 작업 큐에서 Runnable 또는 Callable 객체를 가져와
  * 스레드로 하여금 run()과 call메소드를 실행토록 하는 것이다.



### 작업 처리 요청

* ExecutorService의 작업 큐에 Runnable 또는 Callable 객체를 넣는 행위를 말한다.
* 작업 처리 요청을 위해 ExecutorService는 다음 두 가지 종류의 메소드를 제공한다.
  * ```void execute(Runnable command)```
    * Runnable을 작업 큐에 저장
    * 작업 처리 결과를 받지 못함
  * ```Future<?> submit(Runnable task)```
  * ```Future<V> submit(Runnable task, V result)```
  * ```Future<V> submit(Callable<V> task)```
    * Runnable 또는 Callable을 작업 큐에 저장
    * 리턴 된 Future를 통해 작업 처리 결과 얻을 수 있음
* 작업 처리 도중 예외가 발생할 경우
  * execute()
    * 스레드가 종료되고 해당 스레드는 제거된다. 따라서 스레드 풀은 다른 작업 처리를 위해 새로운 스레드를 생성한다.
  * submit()
    * 스레드가 종료되지 않고 다음 작업을 위해 재사용 된다.(스레드는 재사용하는 것이 좋다. 스레드를 다시 생성하게 되면 CPU가 비효율적인 작업을 하게 된다.)



### 블로킹 방식의 작업 완료 통보 받기

블로킹 방식이라는 것은 무언가를 요청하고 나서, 그 요청에 대한 결과 값이 오기를 기다리는 방식이다.

- ```Future<?> submit(Runnable task)```
- ```Future<V> submit(Runnable task, V result)```
- ```Future<V> submit(Callable<V> task)```
  - Runnable 또는 Callable을 작업 큐에 저장
  - 리턴 된 Future를 통해 작업 처리 결과 얻을 수 있음

* Future

  * 작업 결과가 아니라 지연 완료(pending completion) 객체이다.

  * 작업이 완료될 때까지 기다렸다가 최종 결과를 얻기 위해서 get() 메소드를 사용한다.

  * Future의 메소드 get

    * ```V get()```
      * 작업이 완료될 때까지 블로킹 되었다가 처리 결과 V를 리턴
    * ```V get(long timeout, TimeUnit unit)``` 
      * Timeout 시간동안 작업이 완료되면 결과 V를 리턴하지만, 작업이 완료되지 않으면 TimeoutException을 발생시킨다.

    | 메소드                                   | 작업 처리 완료 후 리턴 타입             | 작업 처리 중 예외 발생         |
    | ------------------------------------- | ---------------------------- | --------------------- |
    | submit(Runnable task)                 | future.get() -> null         | future.get() -> 예외 발생 |
    | submit(Runnable task, Integer result) | future.get() -> Integer 타입 값 | future.get() -> 예외 발생 |
    | submit(Callable\<String > task)       | future.get() -> String 타입 값  | future.get() -> 예외 발생 |

  * Future의 get()은 UI 스레드에 호출하면 안된다.

    * UI를 변경하고 이벤트를 처리하는 스레드가 get() 메소드를 호출하면 작업을 완료하기 전까지는 UI를 변경할 수도 없고 이벤트도 처리할 수 없게 된다.
    * 이것을 해결하려면,
      * 새로운 스레드를 생성해서 호출하거나
      * 스레드풀의 스레드가 호출하도록 해야한다.

  * 다른 메소드

    | 리턴타입    | 메소드명(매개변수)                            | 설명                   |
    | ------- | ------------------------------------- | -------------------- |
    | boolean | cancel(boolean mayInterruptIfRunning) | 작업 처리가 진행중일 경우 취소 시킴 |
    | boolean | isCancelled()                         | 작업이 취소되었는지 여부        |
    | boolean | isDone()                              | 작업 처리가 완료 되었는지 여부    |



### 리턴값이 없는 작업 완료 통보

```java
Runnable task = new Runnable() {
  @Override
  public void run() {
    //스레드가 처리할 작업 내용
  }
};

Future future = executorService.submit(task);
```

```java
try {
  future.get();	//스레드가 작업을 완료할 때 까지 기다린다. 블로킹 된다. 스레드의 작업이 완료되면 블로킹 해제 된다.
} catch (InterruptException e) {
  //작업 처리 도중 스레드가 interrupt 될 경우 실행할 코드
} catch (ExecutionExceprion e) {
  //작업 처리 도중 예외가 발생된 경우 실행할 코드
}
```

​    

### 리턴값이 있는 작업 완료 통보

```java
Callable<T> task = new Callable<T>() {
  @Override
  public T call() throws Exception {
    //스레드가 처리할 작업 내용
    return T;
  }
};

Future<T> future = executorService.submit(task);
```

```java
tyt {
  T result = future.get();  //블로킹 후 작업 완료 시 리턴 값 저장.
} catch (InterruptException e) {
  //작업 처리 도중 스레드가 interrupt 될 경우 실행할 코드
} catch (ExecutionExceprion e) {
  //작업 처리 도중 예외가 발생된 경우 실행할 코드
}
```

  

### 작업 처리 결과를 외부 객체에 저장

스레드가 작업 처리를 완료하고 외부 Result 객체에 작업 결과를 저장하면, 애플리케이션이 Result 객체를 사용해서 어떤 작업을 진행할 수 있을 것이다. 대게 Result 객체는 공유 객체가 되어, 두 개 이상의 스레드 작업을 취합할 목적으로 이용된다. 

```java
Result result = ...;
Runnable task = new Task(result);

Future<Result> future = executorService.submit(task, result);
result = future.get();
```

```java
class Task implements Runnable {
  Result result;
  
  Task(Result result) {
    this.result = result;
  }
  
  @Override
  public void run() {
    //작업 코드
    //처리 결과를 result에 저장
  }
}
```

아래의 예제를 통해서 1부터 10까지 합을 계산하는 두 개의 작업을 스레드 풀에 처리 요청하고, 각각의 스레드가 Result 객체에 결과를 누적하는 결과를 확인할 수 있다. 아래의 코드에서 future1.get() 메소드의 리턴값을 받아 오는 코드를 생략한다면, 블로킹되지 않으므로 결과값은 55 또는 110이 혼재되서 나올 것이다. 작업 완료 후의 값을 가지고 처리하는 작업이라면 get() 메소드를 통해서 꼭 블로킹을 해줘야 한다.

**ResultByRunnableEx.java**

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class ResultByRunnableEx {
  public static void main(String[] args) {
    ExecutorService executorService = Executors.newFixedThreadPool(
        Runtime.getRuntime().availableProcessors()
    );

    System.out.println("[작업 처리 요청]");
    class Task implements Runnable {
      private Result result;

      Task(Result result) {
        this.result = result;
      }

      @Override
      public void run() {
        for (int i = 1; i <= 10; i++) {
          result.addValue(i);
        }
      }
    }

    Result result = new Result();
    Runnable task1 = new Task(result);
    Runnable task2 = new Task(result);

    Future<Result> future1 = executorService.submit(task1, result);
    Future<Result> future2 = executorService.submit(task2, result);

    try {
      result = future1.get();
      result = future2.get();
      System.out.println("[처리 결과] " + result.getAccumValue());
      System.out.println("[작업 처리 완료]");
    } catch (Exception e) {
      e.printStackTrace();
      System.out.println("[실행 예외 발생] " + e.getMessage());
    }
  }
}

class Result {
  private int accumValue = 0;

  synchronized void addValue(int value) {
    accumValue += value;
  }

  public int getAccumValue() {
    return accumValue;
  }
}
```

```java
[작업 처리 요청]
[처리 결과] 110
[작업 처리 완료]
```

  

### 작업 완료순으로 통보 받기

* 작업 요청 순서대로 작업 처리가 완료되는 것은 아니다.
  * 작업의 양과 스레드 스케쥴링에 따라서 먼저 요청한 작업이 나중에 완료되는 경우도 발생한다.
  * 여러 개의 작업들이 순차적으로 처리될 필요성이 없고, 처리결과도 순차적으로 이용할 필요가 없다면,
    * 작업 처리가 완료된 것부터 결과를 얻어 이용하는 것이 좋다.
* 스레드풀에서 작업 처리가 완료된 것만 통보 받는 방법
  * **CompletionService**는 처리 완료된 작업을 가져오는 poll()과 take() 메소드를 제공한다.

| 리턴타입      | 메소드명(매개변수)                        | 설명                                       |
| --------- | --------------------------------- | ---------------------------------------- |
| Future<V> | poll()                            | 완료된 작업의 Future를 가져옴. 완료된 작업이 없다면 즉시 null을 리턴 |
| Future<V> | poll(long timeout, TimeUnit unit) | 완료된 작업의 Future를 가져옴. 완료된 작업이 없다면 timeout까지 블로킹됨 |
| Future<V> | take()                            | 완료된 작업의 Future를 가져옴. 완료된 작업이 없다면 있을 때까지 블로킹됨 |
| Future<V> | submit(Callable<V> task)          | 스레드풀에 Callable 작업 처리 요청                  |
| Future<V> | submit(Runnable task, V result)   | 스레드풀에 Runnable 작업 처리 요청                  |

* CompletionService 객체 얻기

  ```java
  ExecutorService executorService = 
    Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());

  CompletionService<V> completionService = 
    new ExecutorCompletionService<V>(executorService);
  ```

* 작업 처리 요청 방법

  * poll()과 take() 메소드를 이용해서 처리 완료된 작업의 Future를 얻으려면 **CompletionService의 submit() 메소드로 작업 처리 요청을 해야 한다.**

    ```Java
    completionService.submit(Callable<V> task);
    completionService.submit(Runnable<V> task, V result);
    ```

* 완료된 작업 통보 받기

  * take() 메소드를 반복 실행해서 완료된 작업을 계속 통보 받을 수 있도록 한다.

  * 다음은 take() 메소드를 호출하여 완료된 Callable 작업이 있을 때까지 블로킹되었다가 완료된 작업의 Future를 얻고, get() 메소드로 결과값을 얻어내는 코드이다.

  * 아래에서 completionService.take()를 호출하는데 별도도 스레드풀의 스레드에서 호출 한 것은 while문이 애플리케이션이 종료될 때까지 반복 실행해야 하기 때문이다.

  * take() 메소드가 리턴하는 완료된 작업은 submit()으로 처리 요청한 작업의 순서가 아님을 명심해야 한다. 작업의 내용에 따라서 먼저 요청한 작업이 나중에 완료될 수도 있기 때문이다.

    ```java
    executorService.submit(new Runnable() {
      @Override
      public void run() {
        while(true) {
          try {
            Future<Integer> future = completionService.take();
            int value = future.get();
            System.out.println("[처리 결과]" + value);
          } catch (Exception e) {
            break;
          }
        }
      }
    });
    ```


### 콜백 방식의 작업 완료 통보

* 콜백이란

  * 애플리케이션의 스레드에게 작업 처리 요청한 후 다른 기능을 수행할 동안 스레드가 작업을 완료하면 애플리케이션의 메소드를 자동 실행하는 기법을 말한다. 이 때, 자동 실행되는 메소드를 콜백 메소드라고 한다.
  * 블로킹 방식은 작업 처리를 요청한 후 완료될 떄까지 블로킹되지만,
  * 콜백 방식은 작업 처리를 요청한 후 결과를 기다릴 필요 없이 다른 기능을 수행할 수 있다.
  * 그 이유는 작업 처리가 완료되면 자동적으로 콜백 메소드가 실행되어 결과를 알 수 있기 때문이다.

* 콜백 객체와 콜백 하기

  * 콜백 객체 : 콜백 메소드를 가지고 있는 객체

    * Java.nio.channels.CompletionHandler 인터페이스를 활용해서 만들 수도 있다.

    ```java
    CompletionHandler<V, A> callback = new CompletionHandler<V, A> {
      @Override
      public void complited(V result, A attachment) {
          
      }
      
      @Override
      public void failed(Throwable exc, A attachment) {
         
      }
    };
    ```

  * 콜백 하기 : 스레드에서 콜백 객체의 메소드 호출

    * 메소드의 실행은 메인 스레드가 아닌 스레드풀의 스레드에서 호출해야 한다.

    ```java
    Runnable task = new Runnable() {
      @Override
      public void run() {
        try {
          //작업 처리
          V result = ...;
          callback.completed(result, null);
        } catch(Exception e) {
          callback.failed(e, null);
        }
      }
    };
    ```

* [콜백 방식의 작업 완료 통보 실습 예제 소스코드 링크](https://github.com/namjunemy/this_is_java/blob/master/ch12_%EB%A9%80%ED%8B%B0%EC%8A%A4%EB%A0%88%EB%93%9C/src/callback/CallbackEx.java)











