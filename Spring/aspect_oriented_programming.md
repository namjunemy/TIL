# 9. AOP(Aspect Oriented Programming) I

OOP(Object Oriented Programming)언어인 Java로 만들어진 Spring 프레임워크를 공부하고 있다. AOP가 무엇인지 알아보고, XML 기반의 AOP를 구현해본다.

  

## 9-1. AOP란?

관점 지향 프로그램이라고도 한다. 어떤 프로세스가 나열되어 있을 때, 어느 시점을 바라보느냐에 따라서 프로그램이 된다.

프로그래밍을 하다 보면, 공통적인 기능이 많이 발생한다. 이러한 공통 기능은 모든 모듈에 적용하기 위한 방법으로 상속을 통한 방법이 있다. 상속도 좋은 방법이긴 하지만 몇 가지 문제가 있다. 우선 Java에서는 다중상속이 불가능하므로 다양한 모듈에 상속을 통한 공통 기능 부여는 한계가 있다. 그리고, 기능 구현부분에 핵심 기능 코드와 공통 기능 코드가 섞여 있어 효율성이 떨어진다.

이처럼 상속을 통한 방법에 한계가 있어 AOP가 등장하게 되었다. AOP는 핵심 기능과 공통 기능을 분리 시켜놓고, 공통 기능을 필요로 하는 핵심 기능들에서 사용하는 방식이다.

예를 들어서, 아침에 밥을 짓는다고 생각해보자. **핵심 기능**은 쌀을 씻고, 깨끗한 물을 적당히 넣고, 전자밥솥에 내 솥을 넣고, 취사 버튼을 누르는 기능들 일 것이다. **공통 기능**은 수도 꼭지를 열어 물을 받고, 쌀이 깨끗이 씻겨졌는지 눈으로 판단하고, 물이 적당한지 판단하는 기능들 일 것이다. 이러한 기능이 공통 기능인 것은 밥을 짓는 행동이 아닐 떄도 우리는 수도 꼭지를 열고, 눈으로 사물을 보고 적절한 판단을 하기 때문이다. **이렇게 핵심 기능과 공통 기능을 분리해 놓고**, 추후에 밥을 짓는 행동 말고 팥죽을 쑤거나 할 때도 핵심 기능은 변하지만, 공통 기능은 다시 적용할 수 있을 것이다.

AOP 기법이 바로 이런 것이다. 공통 기능을 핵심 기능과 분리해 놓고, 공통 기능 중에서 핵심 기능에 적용하고자 하는 부분에 적용하는 것이다. 그러면서 핵심 기능 실행 중 어느 시점에 공통 기능을 실행할 것에 대한 고민을 하는 것이 AOP 프로그래밍이다.

* AOP방법을 익히려면 우선적으로 조금은 생소한 용어에 익숙해져야 한다.
  * Aspect : 공통 기능
  * Advice : Aspect의 기능 자체
    * Advice종류
      * `<aop:before>` : 메소드 실행 전에 advice 실행
      * `<aop:after-running>` : 정상적으로 메소드 실행 후에 advice 실행
      * `<aop:after-throwing>` : 메소드 실행중 exception 발생시 advice 실행
      * `<aop:after>` : 메소드 실행중 exception이 발생하여도 advice 실행
      * `<aop:around>` : 메소드 실행 전/후 및 exception 발생시 advice 실행
  * JoinPoint : Advice를 적용해야 되는 부분. 즉, 핵심기능들 하나 하나를 joinpoint라고 한다,(ex. 필드, 메소드)(스프링에서는 메소드만 해당)
  * Pointcut : Joinpoint의 부분으로 실제로 Advice가 적용된 부분
  * Weaving : Advice를 핵심 기능(Pointcut)에 적용하는 행위 자체
* 스프링에서 AOP 구현 방법 : Proxy를 이용 한다.
  * 공통 기능(Advice)을 핵심 기능(Pointcut)인 메소드에 삽입을 할 때(Weaving할 때), AOP는 공통기능(Advice)를 직접 삽입하지 않고, Proxy를 이용하여 삽입하고 메소드를 실행한다.
  * 메소드 시작점과 끝점에서 공통기능을 수행해야 하는 상황이라고 가정을 했을 때, 실행 로직을 정확하게 보면 Proxy가 삽입될 위치에서 공통기능을 수행하고 다음 순서로 핵심 기능인 메소드를 수행한다. 그 다음에 다시 Proxy로 돌아와서 공통기능을 수행하고 종료되는 로직이다.
  * 따라서, 호출부인 Client는 핵심기능과 직접 접촉하지 않고 Proxy하고만 접촉을 하면 된다.

## 9-2. 스프링에서 AOP 구현 방식

#### 9-2-1. XML 스키마 기반의 AOP구현

*  작업 순서
  * 의존 설정(pom.xml)
  * 공통 기능의 클래스 제작 - Advice 역할 클래스
  * XML설정 파일에 Aspect 설정

### 기본 개념

* 의존 설정
  * 먼저 pom.xml에 의존 설정을 해준다.
  * aopalliance 라이브러리가 maven repo에 받아진다.

```xml
<!-- AOP -->
<dependency>
  <groupId>org.aspectj</groupId>
  <artifactId>aspectjweaver</artifactId>
  <version>1.6.11</version>
</dependency>
```

* 공통 기능 클래스
  * Proxy(대행자) 역할을 한다.
  * ProceedingJoinPoint(핵심 기능의 부분)을 인자로 받는다.
  * 공통 기능을 수행한 뒤 콘솔 출력을 수행하고,
  * Try 문에서, Proxy의 역할인 joinpoint(핵심 기능)을 실행 시킨다.
  * 그리고 finally로 넘어와서 다시 공통 기능을 수행한다.

```java
import org.aspectj.lang.ProceedingJoinPoint;

public class LogAop {
  public Object loggerAop(ProceedingJoinPoint joinpoint) throws Throwable {
    String signatureStr = joinpoint.getSignature().toShortString();
    System.out.println(signatureStr + "is start");
    long st = System.currentTimeMillis();
    
    try {
      Object obj = joinpoint.proceed();
      return obj;
    } finally {
      long et = System.currentTimeMillis();
      System.out.println(signatureStr + "is finished.");
      System.out.println(signatureStr + " 경과시간 : " + (et - st));
    }
  }
}
```

* XML파일 설정
  * `<bean id="logAop" class="com.javalec.ex.LogAop" />` - 빈을 생성한다.
  * `<aop:config>` - 태그를 이용하여 app설정 정보를 입력한다.
  * `<aop:aspect id="logger" ref="logAop">` - id는 logger이며 참조는 위에 선언된 Bean인 logApp을 참조한다.
  * `<aop:pointcut id="publicM" expression="within(com.javalec.ex.*)" />` - aop의 pointcut 즉, 공통기능을 적용할 핵심기능의 id는 publicM이며 범위(expression)는 com.javalec.ex의 모든 메소드(*)이다.
  * `<aop:aroud pointcut-ref="publicM" method="loggerAop" />` - advice 즉, 공통 기능 타입중 aroud라는 타입을 사용하며 아이디가 publicM인 pointcut을 참조하며, loggerAop메소드에 적용한다.

```xml
<bean id="logApp" class="com.javalec.ex.LogAop" />

<aop:config>
  <aop:aspect id="logger" ref="logAop">
    <aop:pointcut id="publicM" expression="within(com.javalec.ex.*)" />
    <aop:around pointcut-ref="publicM" method="loggerAop" />
  </aop:aspect>
</aop:config>
```

### 실습 코드 분석

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_9_2_ex1_springex)
* LogAop.java

```java
import org.aspectj.lang.ProceedingJoinPoint;

public class LogAop {
  public Object loggerAop(ProceedingJoinPoint joinpoint) throws Throwable {
    String signatureStr = joinpoint.getSignature().toShortString();
    System.out.println(signatureStr + " is start.");
    long st = System.currentTimeMillis();

    try {
      Object obj = joinpoint.proceed();
      return obj;
    } finally {
      long et = System.currentTimeMillis();
      System.out.println(signatureStr + " is finished.");
      System.out.println(signatureStr + " 경과시간 : " + (et - st));
    }
  }
}
```

* Mainclass.java

```java
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.GenericXmlApplicationContext;

public class MainClass {
  public static void main(String[] args) {
    AbstractApplicationContext ctx = new GenericXmlApplicationContext("classpath:applicationCTX.xml");

    Student student = ctx.getBean("student", Student.class);
    student.getStudentInfo();

    Worker worker = ctx.getBean("worker", Worker.class);
    worker.getWorkerInfo();

    ctx.close();
  }
}
```

* applicationCTX.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.2.xsd">

  <bean id="logAop" class="com.javalec.ex.LogAop" />

  <aop:config>
    <aop:aspect id="logger" ref="logAop">
      <aop:pointcut id="publicM" expression="within(com.javalec.ex.*)" />
      <aop:around pointcut-ref="publicM" method="loggerAop" />
    </aop:aspect>
  </aop:config>

  <bean id="student" class="com.javalec.ex.Student">
    <property name="name" value="홍길동" />
    <property name="age" value="10" />
    <property name="gradeNum" value="3" />
    <property name="classNum" value="5" />
  </bean>

  <bean id="worker" class="com.javalec.ex.Worker">
    <property name="name" value="홍길순" />
    <property name="age" value="35" />
    <property name="job" value="개발자" />
  </bean>

</beans>
```

* pom.xml

```xml
...
    <!-- AOP -->
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.6.11</version>
    </dependency>

  </dependencies>
</project>
```

* 결과

```java
Student.getStudentInfo() is start.
이름 : 홍길동
나이 : 10
학년 : 3
반 : 5
Student.getStudentInfo() is finished.
Student.getStudentInfo() 경과시간 : 37
  
Worker.getWorkerInfo() is start.
이름 : 홍길순
나이 : 35
직업 : 개발자
Worker.getWorkerInfo() is finished.
Worker.getWorkerInfo() 경과시간 : 38
```



# 10. AOP(Aspect Oriented Programming) II

## 10-1. @Aspect를 이용한 AOP구현

* 작업순서
  * 의존 설정(pom.xml)
  * @Aspect 어노테이션을 이용한 Aspect클래스 제작
    * Aspect Class 내부에는 @pointcut이 정의되고, Advice의 5가지 종류를 어노테이션으로 사용할 수 있다.
  * XML파일에 \<aop:aspectj-autoproxy /> 설정

### 실습 코드 분석

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_10_1_ex1_springex)


* pom.xml
  * 가장먼저 dependency를 추가하여 AOP에 필요한 라이브러리를 가져온다.

```xml
...
	<!-- AOP -->
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.6.11</version>
    </dependency>

  </dependencies>
</project>
```

* LogAop.java
  * Proxy역할을 하는 Aspect클래스를 만든다. @Aspect 어노테이션 이용
  * AOP의 핵심기능 선언을 해준다. @Pointcut 어노테이션을 이용하여 선언하며, xml파일설정 방법에서 express속성에 해당하는 범위는 com.jacalec.ex패키지의 모든 클래스(메소드)이다.
  * xml파일설정 방법에서 \<aop:around\> 에 해당하는, 실제 Advice의 종류를 어노테이션으로 간단히 추가할 수 있다. 인자로는 "pointcutMethod()"를 넣어주며 바로 위에 정의된 pointcutMethod() 참조하여 핵심 기능의 처음과 끝에 Advice가 적용된다.
  * Pointcut을 먼저 선언하고 Advice를 선언하면서 참조하는 방법이 있고,
  * @Before어노테이션 처럼 바로 사용하는 방법도 있다. 이때는 인자로 바로 within(com.jacalec.ex.Student)라는 적용 범위 설정을 해주면서 Before Advice의 실행 순간은 핵심 기능 실행 이전이다 라는 것을 정의해준다.

```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class LogAop {
  @Pointcut("within(com.javalec.ex.*)")
  private void pointcutMethod() {
  }

  @Around("pointcutMethod()")
  public Object loggerAop(ProceedingJoinPoint joinpoint) throws Throwable {
    String signatureStr = joinpoint.getSignature().toShortString();
    System.out.println(signatureStr + " is start.");
    long st = System.currentTimeMillis();

    try {
      Object obj = joinpoint.proceed();
      return obj;
    } finally {
      long et = System.currentTimeMillis();
      System.out.println(signatureStr + " is finished.");
      System.out.println(signatureStr + " 경과시간 : " + (et - st));
    }
  }

  @Before("within(com.javalec.ex.Student)")
  public void beforAdvice() {
    System.out.println("beforAdvice()");
  }
}
```

* applicationCTX.xml
  * xml파일에서는 `<aop:aspectj-autoproxy />` 라는 설정만 해주면 된다.
  * 그리고 logAop 빈을 생성하면 @Aspect 어노테이션을 이용하여 구현한 AOP가 적용된다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.2.xsd">

  <aop:aspectj-autoproxy />
  <bean id="logAop" class="com.javalec.ex.LogAop" />

  <bean id="student" class="com.javalec.ex.Student">
    <property name="name" value="홍길동" />
    <property name="age" value="10" />
    <property name="gradeNum" value="3" />
    <property name="classNum" value="5" />
  </bean>

  <bean id="worker" class="com.javalec.ex.Worker">
    <property name="name" value="홍길순" />
    <property name="age" value="35" />
    <property name="job" value="개발자" />
  </bean>

</beans>
```

* 그외에 클래스들과 메인메소드는 같고, 실행결과는 다음과 같다.
  * aroud Advice로 감싸진 메소드들은 핵심 기능 앞 뒤로 해당 advice가 적용되었고,
  * before Advice로 감싸진 Student클래스의 메소드는 핵심 기능 전에 advice가 실행되었다.

```java
Student.getStudentInfo() is start.
beforeAdvice()
이름 : 홍길동
나이 : 10
학년 : 3
반 : 5
Student.getStudentInfo() is finished.
Student.getStudentInfo() 경과시간 : 21
Worker.getWorkerInfo() is start.
이름 : 홍길순
나이 : 35
직업 : 개발자
Worker.getWorkerInfo() is finished.
Worker.getWorkerInfo() 경과시간 : 13
```

  

## 10-2. AspectJ Pointcut 표현식

`@Pointcut("within(com.javalec.ex.*)")` 에 사용되는 표현식을 알아본다.

|  문자  |  의미   |
| :--: | :---: |
|  *   |  모든   |
|  .   |  현재   |
|  ..  | 0개 이상 |

* execution
  * @Pointcut("execution(public void get*(..))")
    - public void인 모든 메소드
  * @Pointcut("execution(* com.javalec.ex.\*.\*())")
    * 접근제한자와 반환형 상관없이 com.javalec.ex 패키지에 파라미터가 없는 모든 메소드
  * @Pointcut("execution(* com.javalec.ex..\*.*.())")
    - com.javalec.ex 패키지 & com.javalec.ex 하위 패키지에 파라미터가 없는 모든 메소드
  * @Pointcut("execution(* com.javalec.ex.Worker.*())
    - com.javalec.ex.Worker 클래스 안의 모든 메소드
* within
  * @Pointcut("within(com.javalec.ex.*)")
    * com.javalec.ex 패키지 안에 있는 모든 메소드
  * @Pointcut("within(com.javalec.ex..*)")
    * com.javalec.ex 패키지 및 하위 패키지 안에 있는 모든 메소드
  * @Pointcut("within(com.javalec.ex.Worker)")
    * com.javalec.ex.Worker 모든 메소드
* bean
  * @Pointcut("bean(student)")
    * student 빈에만 적용
  * @Pointcut("bean(*ker)")
    * ~ker로 끝나는 빈에만 적용
* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_10_1_ex1_springex)
  * 위에서 실습한 코드에 Pointcut 표현식만 바꿔서 적용한 뒤 결과를 확인한다.
  * 물론   @Aspect로 구현한 AOP뿐만 아니라 xml 설정 파일로 구현한 AOP에서도 표현식을 적용하여 사용할 수 있다.