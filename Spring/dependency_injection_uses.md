# 5.DI(Dependency Injection)활용

### 5-1. 의존 관계

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_5_1_ex1_springex)


* DI는 Dependency Injection의 약자로 우리말로 '의존 주입'이다.

  * 의존성을 주입하는 방법은 
    * applicationCTX.xml을 통해서 주입하는 방법
    * Java 파일에서 주입하는 방법이 있다.(아래 MainClass.java 참조)

* 예제를 통해서 '의존 주입'이 무엇인지 살펴본다.

  * setter가 아닌 생성자를 통해서 필드의 값을 주입하는 방법
  * property가 아닌 constructor-arg키워드를 이용하여 주입한다.

* src/main/resources/applicationCTX.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="student1" class="com.javalec.ex.Student">
      <constructor-arg>
        <value>홍길동</value>
      </constructor-arg>
      <constructor-arg>
        <value>10살</value>
      </constructor-arg>
      <constructor-arg>
        <value>3학년</value>
      </constructor-arg>
      <constructor-arg>
        <value>20번</value>
      </constructor-arg>
    </bean>

    <bean id="student2" class="com.javalec.ex.Student"> //생성자를 통해서 객체의 값을 주입하는 방법
      <constructor-arg value="홍길동" />
      <constructor-arg value="9살" />
      <constructor-arg value="2학년" />
      <constructor-arg value="10번" />
    </bean>

    <bean id="studentInfo" class="com.javalec.ex.StudentInfo">
      <constructor-arg>
        <ref bean="student1" />
      </constructor-arg>
    </bean>

  </beans>
  ```

* src/main/java/MainClass.java

  * xml 파일에서 studentInfo의 값을 student1으로 먼저 세팅했다.
  * 따라서 첫번째 studentInfo.getStudentInfo(); 에서는 student1의 정보가 출력 될 것이다.
  * 그러나, **Java 파일에서 의존객체를 주입**할 수 있다.
  * ctx.getBean() 메소드로 student2객체를 가져와서 studentInfo setter를 통해 주입해준다.
  * 두번째 studentInfo.getStudentInfo(); 에서는 student2의 정보가 출력 될 것이다.

  ```java
  import org.springframework.context.support.AbstractApplicationContext;
  import org.springframework.context.support.GenericXmlApplicationContext;

  public class MainClass {
    public static void main(String[] args) {
      String configLocatioin = "classpath:applicationCTX.xml";
      AbstractApplicationContext ctx = new GenericXmlApplicationContext(configLocatioin);
      StudentInfo studentInfo = ctx.getBean("studentInfo", StudentInfo.class);
      studentInfo.getStudentInfo();

      Student student2 = ctx.getBean("student2", Student.class);
      studentInfo.setStudent(student2);
      studentInfo.getStudentInfo();

      ctx.close();
    }
  }
  ```

* src/main/java/Student.java

  ```java
  public class Student {
    private String name;
    private String age;
    private String gradeNum;
    private String classNum;

    public Student(String name, String age, String gradeNum, String classNum) {
      this.name = name;
      this.age = age;
      this.gradeNum = gradeNum;
      this.classNum = classNum;
    }

    public String getName() {
      return name;
    }

    public String getAge() {
      return age;
    }

    public String getGradeNum() {
      return gradeNum;
    }

    public String getClassNum() {
      return classNum;
    }

    public void setName(String name) {
      this.name = name;
    }

    public void setAge(String age) {
      this.age = age;
    }

    public void setGradeNum(String gradeNum) {
      this.gradeNum = gradeNum;
    }

    public void setClassNum(String classNum) {
      this.classNum = classNum;
    }
  }
  ```

* src/main/java/StudentInfo.java

  ```java
  public class StudentInfo {
    private Student student;

    public StudentInfo(Student student) {
      this.student = student;
    }

    public void getStudentInfo() {
      if (student != null) {
        System.out.println("이름 : " + student.getName());
        System.out.println("나이 : " + student.getAge());
        System.out.println("학년 : " + student.getGradeNum());
        System.out.println("반 : " + student.getClassNum());
        System.out.println("======================");
      }
    }

    public void setStudent(Student student) {
      this.student = student;
    }
  }

  ```

* 출력결과

  ```java
  이름 : 홍길동
  나이 : 10살
  학년 : 3학년
  반 : 20번
  ======================
  이름 : 홍길동
  나이 : 9살
  학년 : 2학년
  반 : 10번
  ======================
  ```

  

### 5-2. DI 사용에 따른 장점

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_5_2_ex1_springex)

사실 작은 규모의 프로젝트에서는 스프링의 DI를 사용하는 것 보다 일반적인 방법을 사용하여 개발하는 것이 더욱 빠르고, 개발에 따른 스트레스를 줄일 수 있다. 하지만 규모가 어느 정도 커지고, 추후 유지보수 업무가 발생할 때 DI를 이용한 개발의 장점을 느낄 수 있다. 예제를 통해 살펴보자.

* Java 파일의 수정 없이 스프링 설정 파일만을 수정하여 부품들을 생성/조립할 수 있다.

* src/main/java/MainClass.java

  ```java
  import org.springframework.context.support.AbstractApplicationContext;
  import org.springframework.context.support.GenericXmlApplicationContext;

  public class MainClass {
    public static void main(String[] args) {
      AbstractApplicationContext ctx = new GenericXmlApplicationContext("classpath:applicationCTX.xml");
      Pencil pencil = ctx.getBean("pencil", Pencil.class);
      pencil.use();
      
      ctx.close();
    }
  }
  ```

* src/main/java/Pencil.java

  ```java
  public interface Pencil {
    public void use();
  }
  ```

* src/main/java/Pencil4B.java

  ```java
  public class Pencil4B implements Pencil {
    @Override
    public void use() {
      System.out.println("4B 굵기로 쓰입니다.");
    }
  }
  ```

* src/main/java/Pencil6B.java

  ```java
  public class Pencil6B implements Pencil {
    @Override
    public void use() {
      System.out.println("6B 굵기로 쓰입니다.");
    }
  }
  ```

* src/main/java/Pencil6BWithEraser.java

  ```java
  public class Pencil6BWithEraser implements Pencil {
    @Override
    public void use() {
      System.out.println("6B 굵기로 쓰이고, 지우개가 있습니다.");
    }
  }
  ```

* src/main/resources/applicationCTX.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="pencil" class="com.javalec.ex.Pencil6BWithEraser"></bean>

  </beans>
  ```

* 위의 예제에서 java 파일들은 건드리지 않고, 스프링 설정 파일인 applicationCTX.xml 파일에서 bean의 class만 수정하면서 프로젝트 소스코드를 수정할 수 있다. 이것이 현재 학습 단계에서 느껴 볼 수 있는 DI 사용에 따른 장점이다.

  ​