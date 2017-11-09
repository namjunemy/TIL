# 6. DI 설정 방법

### 6-1. XML파일을 이용한 DI 설정 방법

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_6_1_ex1_springex)


* XML 파일을 이용한 DI설정 방법은 그 동안 살펴본 방식이다.
* 아래와 같이 생성자를 통해서 필드값을 주입하는 방법이 있고, 클래스의 setter를 이용해서 property값을 할당하는 방법도 있었다.

```xml
<bean id="student1" class="com.javalec.ex.Student">
  <constructor-arg value="홍길동" />
  <constructor-arg value="10" />
  <constructor-arg>
    <list>
      <value>수영</value>
      <value>요리</value>
    </list>
  </constructor-arg>

  <property name="height">
    <value>187</value>
  </property>

  <property name="weight" value="84" />
</bean>

<bean id="sutudentInfo1" class="com.javalec.ex.StudentInfo">
  <property name="student">
    <ref bean="student1" />
  </property>
</bean>
```

* xml에서 새롭게 namespace를 사용하여 필드에 값을 할당하는 방법도 있다.
  * 아래에 family객체의 필드 값을 할당하는 쪽을 보면 c:, p:라는 namespace를 통해서 값을 할당 한 것을 볼 수 있다.
  * c:는 constructor-arg키워드를 사용해서 value를 할당한 것과 같고
  * p:는 property키워드를 사용하여 value를 할당한 것과 같다.
  * 단, namespace를 사용하려면 beans 태그 안에 c와 p에 대한 xmlns를 선언해주면 된다.
  * 이런 방법도 있다는 것만 알아두자. 개인의 취향대로 사용하면 된다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:c="http://www.springframework.org/schema/c"
  xmlns:p="http://www.springframework.org/schema/p"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="student3" class="com.javalec.ex.Student">
    <constructor-arg value="홍길자" />
    <constructor-arg value="8" />
    <constructor-arg>
      <list>
        <value>줄넘기</value>
        <value>공기놀이</value>
      </list>
    </constructor-arg>

    <property name="height">
      <value>126</value>
    </property>

    <property name="weight" value="21" />
  </bean>

  <bean id="family" class="com.javalec.ex.Family" c:papaName="홍아빠" c:mamiName="홈엄마" p:sisterName="홍누나">
    <property name="brotherName" value="홍오빠" />
  </bean>

</beans>
```

* 스프링 설정 파일이 다수인 경우

  * MainClass.java
  * 아래와 같이 GenericXmlApplicationContext를 생성하는 과정에서 연속적으로 넘겨주면 된다.

  ```java
  ...
  String configLocation1 = "classpath:applicationCTX.xml";
  String configLocation2 = "classpath:applicationCTX1.xml";
  AbstractApplicationContext ctx = new GenericXmlApplicationContext(configLocation1, configLocation2);

  Student student1 = ctx.getBean("student1", Student.class);
  ..
  ```

  

### 6-2. Java를 이용한 DI 설정 방법

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_6_2_ex1_springex)


* xml파일로 설정하던 것들을 java의 annotation을 사용하여 할 수 있다.
* **@Configuration** : '스프링 설정에 사용되는 클래스'임을 명시

```java
@Configuration
public class ApplicationConfig {
  ...
}
```

* **@Bean** : 객체 생성

```java
@Bean
public Student student01 {
  ArrayList<String> hobbys = new ArrayLiust<>();
  hobbys.add("수영");
  hobbys.add("수영");
  
  Student student = new Student("홍길동", 20, hobbys);	//생성자에 설정
  student.setHeight(180);	//프로퍼티에 설정
  student.setWeight(80);
  
  return student;
}
```

* ApplicationConfig.java

```java
package com.javalec.ex;

import java.util.ArrayList;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ApplicationConfig {
  @Bean
  public Student student1() {
    ArrayList<String> hobbys = new ArrayList<String>();
    hobbys.add("수영");
    hobbys.add("요리");

    Student student = new Student("홍길동", 20, hobbys);
    student.setHeight(180);
    student.setWeight(80);

    return student;
  }

  @Bean
  public Student student2() {
    ArrayList<String> hobbys = new ArrayList<String>();
    hobbys.add("독서");
    hobbys.add("음악감상");

    Student student = new Student("홍길순", 18, hobbys);
    student.setHeight(170);
    student.setWeight(55);

    return student;
  }
}
```

* MainClass.java
  * Java class를 이용해서 spring config를 해주고 있으므로, AnnotationConfigApplicationContext를 사용한다.

```java
package com.javalec.ex;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class MainClass {
  public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(ApplicationConfig.class);

    Student student1 = ctx.getBean("student1", Student.class);
    System.out.println("이름 : " + student1.getName());
    System.out.println("나이 : " + student1.getAge());
    System.out.println("취미 : " + student1.getHobbys());
    System.out.println("신장 : " + student1.getHeight());
    System.out.println("몸무게 : " + student1.getWeight());

    Student student2 = ctx.getBean("student2", Student.class);
    System.out.println("이름 : " + student2.getName());
    System.out.println("나이 : " + student2.getAge());
    System.out.println("취미 : " + student2.getHobbys());
    System.out.println("신장 : " + student2.getHeight());
    System.out.println("몸무게 : " + student2.getWeight());

    ctx.close();
  }
}
```

  

### 6-3. XML과 Java를 같이 사용

* XML파일에 Java파일을 포함시켜 사용하는 방법

  * 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_6_3_ex1_springex)
  * ApplicationConfig.java

  ```java
  import java.util.ArrayList;

  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;

  @Configuration
  public class ApplicationConfig {
    @Bean
    public Student student1() {
      ArrayList<String> hobbys = new ArrayList<String>();
      hobbys.add("수영");
      hobbys.add("요리");

      Student student = new Student("홍길동", 20, hobbys);
      student.setHeight(180);
      student.setWeight(80);

      return student;
    }
  }

  ```

  * applicationCTX.xml
    * context namespace를 활용하여 annotation-config파일을 Bean으로 생성한다.

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
  		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.2.xsd">

    <context:annotation-config />
    <bean class="com.javalec.ex.ApplicationConfig" />

    <bean id="student2" class="com.javalec.ex.Student">
      <constructor-arg value="홍길순"></constructor-arg>
      <constructor-arg value="30"></constructor-arg>
      <constructor-arg>
        <list>
          <value>마라톤</value>
          <value>요리</value>
        </list>
      </constructor-arg>
      <property name="height" value="190" />
      <property name="weight" value="70" />
    </bean>

  </beans>
  ```

  

* Java파일에 XML파일을 포함시켜 사용하는 방법

  * 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_6_3_ex2_springex)
  * ApplicationConfig.java
    * ImportResource annotation을 이용해서 위의 경우와 반대로 xml파일을 java클래스로 가져온다.

  ```java
  import java.util.ArrayList;

  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.context.annotation.ImportResource;

  @Configuration
  @ImportResource("classpath:applicationCTX.xml")
  public class ApplicationConfig {
    @Bean
    public Student student1() {

      ArrayList<String> hobbys = new ArrayList<String>();
      hobbys.add("수영");
      hobbys.add("요리");

      Student student = new Student("홍길동", 20, hobbys);
      student.setHeight(180);
      student.setWeight(80);

      return student;
    }
  }
  ```

  * applicationCTX.xml

  ```xml
  <beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="student2" class="com.javalec.ex.Student">
      <constructor-arg value="홍길순"></constructor-arg>
      <constructor-arg value="30"></constructor-arg>
      <constructor-arg>
        <list>
          <value>마라톤</value>
          <value>요리</value>
        </list>
      </constructor-arg>
      <property name="height" value="190" />
      <property name="weight" value="70" />
    </bean>

  </beans>
  ```

  ​