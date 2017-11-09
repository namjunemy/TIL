# 7. 생명주기(Life Cycle) 와 범위(Scope)

### 7-1. 스프링 컨테이너 생명 주기

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_7_1_ex1_springex)


* 1. 스프링 컨테이너 생성

  ```java
  GenericXmlApplicationContext ctx = new GenericXmlApplicationContext();

  또는 

  GenericXmlApplicationContext ctx = new GenericXmlApplicationContext("classpath:applicationCTX.xml");
  ```

* 2. 스프링 컨테이너 설정

  * 스프링 컨테이너를 생성할 때, 생성자를 통해 생성했다면 refresh() 메소드를 호출 해 줄 필요가 없지만,
  * ctx의 load() 메소드를 통해서 생성했을 경우, refresh() 메소드를 꼭 호출 해야 한다.

  ```java
  ctx.load("classpath:applicationCTX.xml");

  ctx.refresh();
  ```

* 3. 스프링 컨테이너 사용

  ```java
  Student student = ctx.getBean("student", Student.class);
  System.out.println("이름: " + student.getName());
  System.out.println("나이: " + student.getAge());
  ```

* 4. 스프링 컨테이너 종료

  ```java
  ctx.close();
  ```

  

### 7-2. 스프링 빈 생명 주기

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_7_2_ex1_springex)


* 이번에는 스프링 빈의 생명 주기를 알아본다.
* 빈 초기화 과정과 빈 소멸 과정에서 호출되는 메소드를 알아본다.

```java
public class MainClass {
  public static void main(String[] args) {
    GenericXmlApplicationContext ctx = new GenericXmlApplicationContext();
    
    ctx.load("classpath:applicationCTX.xml");
    
    ctx.refresh();
    
    ctx.close();
  }
}
```

#### 7-2-1. implements InitializingBean, DisposableBean

* 위의 메인 클래스에서 `ctx.refresh()` 메소드가 호출되면서 빈 이 초기화 될 때, 아래의 `afterPropertiesSet()` 메소드가 호출된다.

* 초기화 과정중 해야 할 작업이 있다면, 이 메소드에서 하면 된다.

  ```java
  @Override
  public void afterPropertiesSet() throws Exception {
    ...
  }
  ```

* 메인 클래스에서  `cts.close()` 메소드가 호출되면서 빈이 소멸 될때, 아래의 `destroy()` 메소드가 호출된다. 마찬가지로 빈이 소멸하는 과정중에서 해야 할 작업이 있다면, 이 메소드에서 해주면 된다.

  ```java
  @Override
  public void destroy() throws Exception {
    ...
  }
  ```

> ctx.close() 의 경우 컨테이너가 소멸 하는 단계이다. 컨테이너가 소멸 하면, 빈은 자동 소멸 된다. 빈만 소멸하게 해야 하는 경우 student.destroy() API를 이용하면 된다.

* **InitializingBean, DisposableBean을 구현상속 한 Student 클래스**

```java
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;

public class Student implements InitializingBean, DisposableBean {
  private String name;
  private int age;

  public Student(String name, int age) {
    this.name = name;
    this.age = age;
  }

  public String getName() {
    return name;
  }

  public int getAge() {
    return age;
  }

  @Override
  public void destroy() throws Exception {
    System.out.println("destory()");
  }

  @Override
  public void afterPropertiesSet() throws Exception {
    System.out.println("afterPropertiesSet()");
  }
}
```

  

#### 7-2-2. annotation 을 이용하는 방법

* 위에서는 구현상속을 통해서 메소드를 Override하는 방식이었고, 어노테이션을 사용하는 방법도 있다.

* 위의 메인 클래스에서 `ctx.refresh()` 메소드가 호출되면서 빈 이 초기화 될 때, 아래의 `@PostConstruct` 어노테이션이 선언된 메소드가 호출 된다. 메소드 명은 자유롭게 정의할 수 있다.

  ```java
  @PostConstruct
  public void initMethod() {
    System.out.println("initMethod()");
  }
  ```

* 위의 메인 클래스에서 `ctx.close()` 메소드가 호출되면서 빈이 소멸 될 때, 아래의 `@preDestroy` 어노테이션이 선언된 메소드가 호출 된다. 마찬가지로 메소드 명은 자유롭게 정의할 수 있다.

  ```java
  @PostConstruct
  public void destroyMethod() {
    System.out.println("destroyMethod()");
  }
  ```

* **@PostConstruct, @PreDestroy 어노테이션이 정의 된 OtherStudent 클래스**

```java
import javax.annotation.*;

public class OtherStudent {
  private String name;
  private int age;

  public OtherStudent(String name, int age) {
    this.name = name;
    this.age = age;
  }

  public String getName() {
    return name;
  }

  public int getAge() {
    return age;
  }

  @PostConstruct
  public void initMethod() {
    System.out.println("initMethod()");
  }

  @PreDestroy
  public void destroyMethod() {
    System.out.println("destroyMethod()");
  }
}
```

  

### 7-3. 스프링 빈 범위(Scope)

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_7_3_ex1_springex)


* 스프링 컨테이너가 생성되고, 스프링 빈이 생성 될 때, 생성된 스프링 빈은 scope를 가지고 있다.
* scope랑 쉽게 생각해서 해당하는 객체가 어디까지 영향을 미치는지 결정하는 것이라고 생각하면 된다.
* applicationCTX.xml
  * 아래의 xml에서 bean의 scope가 singleton으로 지정되어 있다. singleton객체는 단 하나만 생성될 수 있는 객체이다. bean의 scope default값은 singleton이다

```Xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="student" class="com.javalec.ex.Student" scope="singleton">
    <constructor-arg value="홍길동"></constructor-arg>
    <constructor-arg value="26"></constructor-arg>
  </bean>
  
</beans>
```

* MainClass.java
  * 아래의 메인 클래스에서 보면 applicationCTX.xml 즉, 스프링 config 파일을 통해서 스프링 컨테이너를 생성하고, getBean() 메소드를 통해서 Student 빈을 가져다 쓴다.
  * student2에 빈을 할당할 때도 보면, 새로이 객체를 생성하는 것이 아니라 이미 생성된 객체를 스프링 컨테이너에서 가져와서 setter로 값을 변경해서 사용하고 있다.
  * 자바의 메모리 구조상으로 봤을 때, student1과 student2가 하나의 Student객체를 바라보고 있는 형태가 될 것이다.
  * 맨 아래에서 두 객체가 같은 객체인 것을 직접 확인 해 볼 수 있다.

```java
public class MainClass {

  public static void main(String[] args) {
    AbstractApplicationContext ctx = new GenericXmlApplicationContext("classpath:applicationCTX.xml");

    Student student1 = ctx.getBean("student", Student.class);
    System.out.println("이름: " + student1.getName());
    System.out.println("나이: " + student1.getAge());

    Student student2 = ctx.getBean("student", Student.class);
    student2.setName("홍길순");
    student2.setAge(23);
    System.out.println("이름: " + student2.getName());
    System.out.println("나이: " + student2.getAge());

    if (student1.equals(student2))
      System.out.println("student1 == student2");
    else
      System.out.println("student1 != student2");

    ctx.close();
  }
}
```

```java
이름: 홍길동
나이: 26
이름: 홍길순
나이: 23
student1 == student2
```

* Student.java

```java
public class Student {
  private String name;
  private int age;

  public Student(String name, int age) {
    this.name = name;
    this.age = age;
  }

  public String getName() {
    return name;
  }

  public int getAge() {
    return age;
  }

  public void setName(String name) {
    this.name = name;
  }

  public void setAge(int age) {
    this.age = age;
  }
}
```

