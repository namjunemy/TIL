# 8. 외부 파일을 이용한 설정

우리가 어떤 스프링 프로젝트를 진행할 때, 프로젝트에 필요한 자바 코드나 설정파일이 있을텐데(예를 들면, db접근 정보, db 계정 정보, 위치 등) 이러한 정보들을 프로젝트 코드 내에 가지고 있는 것이 아니라, 프로젝트가 구동되는 시점에 외부파일에서 정보를 가져와서 사용한다. 만약, 설정 정보들이 변경되었을 경우에 프로젝트 내부 코드가 아닌 외부 설정 파일의 값만 수정함으로써 문제를 해결할 수 있다.

## Environment 객체

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_8_1_ex1_springex)


* Environment 객체를 이용해서 스프링 빈 설정을 한다.
* Spring Context 파일로 부터 getEnvironment() 메소드를 이용하여 Environment 객체를 얻을 수 있다.
* Environment 객체의 getPropertySources() 메소드를 이용하여 각 PropertySource 객체들(여러가지 설정에 관련된 값들)을 가져올 수 있다. 
* 즉, 아래의 구조와 같이 Spring Context 안에 설정에 관련된 값을 얻어 올 수 있는 Environment 객체가 존재하며, 이 객체 하위에는 실제 설정 값에 해당하는 PropertySource 들이 위치하게 된다.
* 실제로 어떤 정보가 필요해서 요청을 하면, Environment객체는 그 안에 있는 PropertySource들을 처음부터 끝까지 쭉 스캔해서 정보가 있는 경우 리턴한다.

![](https://github.com/namjunemy/TIL/blob/master/Spring/img/Environment_01.png?raw=true)

* 위와 같이 propertySources.addLast() 메소드를 통해서 필요한 프로퍼티를 맨 마지막에 추가할 수 있고,
* env.getProperty() 메소드를 통해서 필요한 프로퍼티를 얻을 수도 있다.

#### 실습코드 해석

* AdminConnection.java
  * EnviromentAware, InitializingBean, DisposableBean 3가지 인터페이스를 구현상속한다.
  * 3가지 인터페이스를 구현상속 함으로써 빈을 생성할 때 각각 setEnvironment(), afterPropertiesSet(), destroy() 메소드가 콜백으로 호출된다.
  * setEnvironment()를 통해서 Environment를 빈의 내부 필드에 저장한다.
  * afterPropertiesSet()를 통해서 빈 내부의 Environment 객체로 부터 getProperty() 메소드를 이용하여 property를 가져와서 저장한다.
  * destroy()를 통해서 빈이 소멸 될 때 필요한 작업을 정의할 수 있다.

```java
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.context.EnvironmentAware;
import org.springframework.core.env.Environment;

public class AdminConnection implements EnvironmentAware, InitializingBean, DisposableBean {
  private Environment env;
  private String adminId;
  private String adminPw;

  @Override
  public void setEnvironment(Environment env) {
    System.out.println("setEnvironment()");
    setEnv(env);
  }

  public void setEnv(Environment env) {
    this.env = env;
  }

  public void setAdminId(String adminId) {
    this.adminId = adminId;
  }

  public void setAdminPw(String adminPw) {
    this.adminPw = adminPw;
  }

  public String getAdminId() {
    return adminId;
  }

  public String getAdminPw() {
    return adminPw;
  }

  @Override
  public void afterPropertiesSet() throws Exception {
    System.out.println("afterPropertiesSet()");
    setAdminId(env.getProperty("admin.id"));
    setAdminPw(env.getProperty("admin.pw"));
  }

  @Override
  public void destroy() throws Exception {
    System.out.println("destroy()");
  }
}
```

* admin.properties 파일

```properties
admin.id=abcde
admin.pw=12345
```

* applicationCTX.xml 파일
  * xml파일에서 Bean을 생성한다. Value 주입은 없다. 빈이 초기화 될 때, 인터페이스의 구현상속으로 호출되는 콜백메소드를 통해 내부적으로 값을 세팅한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="adminConnection" class="com.javalec.ex.AdminConnection" />

</beans>
```

* MainClass.java
  * ConfigurableApplicationContext 변수에 GenericXmlApplicationContext를 저장한다.
  * Context객체(ctx)의 getEnvironment() 메소드를 통해 Environment 객체를 변수에 저장한다.
  * Environment객체(env)의 getPropertySources()를 통해 PropertySource를 얻는다.
  * propertySources의 addLast() 메소드를 이용하여 admin.properties에 정의된 Properties값들을 주입한다.(new ResourcePropertySource("admin.properties"));
  * 콘솔 출력을 통해 env.getProperty()를 사용하여 값을 찍어보면 추가된 프로퍼티 값을 얻을 수 있다.
  * 다음으로 맨처음 생성한 Context객체(ctx)를 GenericXmlApplicationContext 변수에 저장하고
  * applicationCTX.xml 파일에 저장된 정보를 로딩한다.
  * context파일로 부터 adminConnection 빈을 가져와서 사용할 수 있다.

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.GenericXmlApplicationContext;
import org.springframework.core.env.ConfigurableEnvironment;
import org.springframework.core.env.MutablePropertySources;
import org.springframework.core.io.support.ResourcePropertySource;

public class MainClass {
  public static void main(String[] args) {
    ConfigurableApplicationContext ctx = new GenericXmlApplicationContext();
    ConfigurableEnvironment env = ctx.getEnvironment();
    MutablePropertySources propertySources = env.getPropertySources();
    
    try {
      propertySources.addLast(new ResourcePropertySource("classpath:admin.properties"));
      
      System.out.println(env.getProperty("admin.id"));
      System.out.println(env.getProperty("admin.pw"));
    } catch (Exception e) {
    }
    
    GenericXmlApplicationContext gCtx = (GenericXmlApplicationContext) ctx;
    gCtx.load("applicationCTX.xml");
    gCtx.refresh();
    
    AdminConnection adminConnection = gCtx.getBean("adminConnection", AdminConnection.class);
    System.out.println("admin ID: " + adminConnection.getAdminId());
    System.out.println("admin PW: " + adminConnection.getAdminPw());
    
    gCtx.close();
    ctx.close();
  }
}
```

  

## 프로퍼티 파일을 이용한 설정

이번에는 Environment객체를 사용하지 않고 프로퍼티 파일을 직접 이용하여 스프링 빈을 설정하는 방법에 대해서 살펴 본다.

#### 스프링 설정 XML파일에 프로퍼티 파일을 명시한다.

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_8_2_ex1_springex)
* 실습 코드 분석


* applicationCTX.xml
  * xmlns:context 네임스페이스를 추가하고, context:property-placeholder 선언을 통해 가져올 properties 파일의 경로를 설정한다.
  * 아래의 Bean생성 시점에서 Property를 할당할 때, value로 값을 직접 넣어줄 수 있다.
  * 이 때, $를 사용하여 { }안에 사용할 변수를 넣어주면 된다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.2.xsd">

  <context:property-placeholder location="classpath:admin.properties, classpath:sub_admin.properties" />

  <bean id="adminConnection" class="com.javalec.ex.AdminConnection">
    <property name="adminId">
      <value>${admin.id}</value>
    </property>
    <property name="adminPw">
      <value>${admin.pw}</value>
    </property>
    <property name="sub_adminId">
      <value>${sub_admin.id}</value>
    </property>
    <property name="sub_adminPw">
      <value>${sub_admin.pw}</value>
    </property>
  </bean>

</beans>
```

* admin.properties

```properties
admin.id=abcde
admin.pw=12345
```

* sub_admin.properties

```properties
sub_admin.id=fghij
sub_admin.pw=67890
```

* AdminConnection.java

```java
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;

public class AdminConnection implements InitializingBean, DisposableBean {
  private String adminId;
  private String adminPw;
  private String sub_adminId;
  private String sub_adminPw;

  public void setAdminId(String adminId) {
    this.adminId = adminId;
  }

  public void setAdminPw(String adminPw) {
    this.adminPw = adminPw;
  }

  public void setSub_adminId(String sub_adminId) {
    this.sub_adminId = sub_adminId;
  }

  public void setSub_adminPw(String sub_adminPw) {
    this.sub_adminPw = sub_adminPw;
  }

  public String getAdminId() {
    return adminId;
  }

  public String getAdminPw() {
    return adminPw;
  }

  public String getSub_adminId() {
    return sub_adminId;
  }

  public String getSub_adminPw() {
    return sub_adminPw;
  }

  @Override
  public void afterPropertiesSet() throws Exception {
    System.out.println("afterPropertiesSet()");
  }

  @Override
  public void destroy() throws Exception {
    System.out.println("destroy()");
  }
}
```

- Mainclass.java

```java
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.GenericXmlApplicationContext;

public class MainClass {

  public static void main(String[] args) {
    AbstractApplicationContext ctx = new GenericXmlApplicationContext("classpath:applicationCTX.xml");

    AdminConnection connection = ctx.getBean("adminConnection", AdminConnection.class);
    System.out.println("adminID : " + connection.getAdminId());
    System.out.println("adminPW : " + connection.getAdminPw());
    System.out.println("sub_adminID : " + connection.getSub_adminId());
    System.out.println("sub_adminPW : " + connection.getSub_adminPw());

    ctx.close();
  }
}
```

```java
afterPropertiesSet()
adminID : abcde
adminPW : 12345
sub_adminID : fghij
sub_adminPW : 67890
destroy()
```

  

#### 스프링 설정 JAVA파일에 프로퍼티 파일을 명시한다.

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_8_2_ex2_springex)
* 실습 코드 분석
* ApplicationConfig.java
  * 6절에서 학습했던것 처럼 ApplicationConfig.java파일을 생성한다.
  * 먼저 첫번째 @Bean 을 보면 바로 위에서 학습했던 xml파일에 properties 파일들을 명시하는 **context:property-placeholder** 네임스페이스에 해당하는 내용이 있다.
  * Properties() 메소드를 만들어주면서 locations 배열을 만들어서 Properties 파일들을 저장하고, PropertySourcesPlaceholderConfigurer에 setLocation() 을 해준뒤에 리턴한다.
  * 다음으로 AdminConnection Bean을 생성하고, setter를 통해서 값을 넣어준다.
  * 주의할점! ApplicationConfig 파일에서는 지금 properties파일의 위치를 지정해주기만 했지 값을 직접 넣어주지는 않았다.
  * 파일의 맨 위에 있는 필드 선언부에 보면 **@Value** 어노테이션이 값을 필드에 set 하는 역할을 한다.
  * 따라서 우리는 별도의 작업 없이 adminConnection의 setter를 통해 필드값을 주입할 수 있다.

```java
import org.springframework.beans.factory.annotation.Value
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;

@Configuration
public class ApplicationConfig {
  @Value("${admin.id}")
  private String adminId;
  @Value("${admin.pw}")
  private String adminPw;
  @Value
  private String sub_adminId;
  @Value
  private String sub_adminPw;
  
  @Bean
  public static PropertySourcesPlaceholderConfigurer Properties() {
    PropertySourcesPlaceholderConfigurer configurer = new PropertySourcesPlaceholderConfigurer();
    
    Resource[] locations = new Resource[2];
    locations[0] = new ClassPathResource("admin.properties");
    locations[1] = new ClassPathResource("sub_admin.properties");
    configurer.setLocations(locations);
    
    return configurer;
  }
  
  @Bean
  public AdminConnection adminConfig() {
    AdminConnection adminConnection = new AdminConnection();
    adminConnection.setAdminId(adminId);
    adminConnection.setAdminPw(adminPw);
    adminConnection.setSub_adminId(sub_adminId);
    adminConnection.setSub_adminPw(sub_adminPw);
    return adminConnection;
  }
}
```

* admin.properties

```properties
admin.id=abcde
admin.pw=12345
```

- sub_admin.properties

```properties
sub_admin.id=fghij
sub_admin.pw=67890
```

- AdminConnection.java

```java
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;

public class AdminConnection implements InitializingBean, DisposableBean {
  private String adminId;
  private String adminPw;
  private String sub_adminId;
  private String sub_adminPw;

  public void setAdminId(String adminId) {
    this.adminId = adminId;
  }

  public void setAdminPw(String adminPw) {
    this.adminPw = adminPw;
  }

  public void setSub_adminId(String sub_adminId) {
    this.sub_adminId = sub_adminId;
  }

  public void setSub_adminPw(String sub_adminPw) {
    this.sub_adminPw = sub_adminPw;
  }

  public String getAdminId() {
    return adminId;
  }

  public String getAdminPw() {
    return adminPw;
  }

  public String getSub_adminId() {
    return sub_adminId;
  }

  public String getSub_adminPw() {
    return sub_adminPw;
  }

  @Override
  public void afterPropertiesSet() throws Exception {
    System.out.println("afterPropertiesSet()");
  }

  @Override
  public void destroy() throws Exception {
    System.out.println("destroy()");
  }
}
```

- MainClass.java

```java
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.GenericXmlApplicationContext;

public class MainClass {
  public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(ApplicationConfig.class);
    AdminConnection connection = ctx.getBean("adminConfig", AdminConnection.class);

    System.out.println("adminID : " + connection.getAdminId());
    System.out.println("adminPW : " + connection.getAdminPw());
    System.out.println("sub_adminID : " + connection.getSub_adminId());
    System.out.println("sub_adminPW : " + connection.getSub_adminPw());

    ctx.close();
  }
}
```

```java
afterPropertiesSet()
adminID : abcde
adminPW : 12345
sub_adminID : fghij
sub_adminPW : 67890
destroy()
```

  

## 프로파일(profile) 속성을 이용한 설정

동일한 스프링 빈을 여러 개 만들어 놓고 상황(환경)에 따라서 적절한 스프링 빈을 사용할 수 있다. Profile 속성을 사용하면 된다.

* 예를 들면, 개발환경에서 필요한 설정파일이 존재할 수 있고, 실제 운영환경에서 필요한 설정파일이 존재할 수 있다. 이런경우에 미리 만들어놓고 적절히 사용할 수 있다.

#### xml 설정 파일을 이용하는 경우

* **자주 쓰인다.**


* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_8_3_ex1_springex)
* 코드 분석
* applicationCTX_dev.xml 파일
  * namespace에 profile만 설정해주면 된다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd"
  profile="dev">

  <bean id="serverInfo" class="com.javalec.ex.ServerInfo">
    <property name="ipNum" value="localhost"></property>
    <property name="portNum" value="8181"></property>
  </bean>

</beans>
```

* applicationCTX_run.xml 파일
  * namespace에 profile만 설정해주면 된다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd"
  profile="run">

  <bean id="serverInfo" class="com.javalec.ex.ServerInfo">
    <property name="ipNum" value="213.186.229.29"></property>
    <property name="portNum" value="80"></property>
  </bean>

</beans>
```

* ServerInfo.java

```java
public class ServerInfo {
  private String ipNum;
  private String portNum;

  public String getIpNum() {
    return ipNum;
  }

  public void setIpNum(String ipNum) {
    this.ipNum = ipNum;
  }

  public String getPortNum() {
    return portNum;
  }

  public void setPortNum(String portNum) {
    this.portNum = portNum;
  }
}
```

* MainClass.java
  * scanner로 config를 입력받아서 값을 설정한다.
  * 받은 config문자열을 ctx의 getEnvieonment객체의 setActiveProfiles(config)를 사용하여 Active시킬 Profile을 설정한다.
  * 설정한 Profile에 해당하는 xml파일로 부터 Bean을 가져와서 사용한다.

```java
import java.util.Scanner;

import org.springframework.context.support.GenericXmlApplicationContext;

public class MainClass {
  public static void main(String[] args) {
    String config = null;
    Scanner scanner = new Scanner(System.in);
    String str = scanner.next();
    if (str.equals("dev")) {
      config = "dev";
    } else if (str.equals("run")) {
      config = "run";
    }
    
    scanner.close();
    
    GenericXmlApplicationContext ctx = new GenericXmlApplicationContext();
    ctx.getEnvironment().setActiveProfiles(config);
    ctx.load("applicationCTX_dev.xml", "applicationCTX_run.xml");
    
    ServerInfo info = ctx.getBean("serverInfo", ServerInfo.class);
    System.out.println("ip : " + info.getIpNum());
    System.out.println("port : " + info.getPortNum());
    ctx.close();
  }
}
```

```java
> run

ip : 213.186.229.29
port : 80
```



#### JAVA 설정 파일을 이용하는 경우

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_8_3_ex2_springex)
* 코드 분석
  * 위의 xml파일을 이용하는 경우와 별다른 차이가 없다. java 파일로 바뀐것 뿐이다.
* ApplicationConfigDev.java
  * @configuration과 @Profile 어노테이션 설정을 해주고  Profile 값을 설정한다.
  * @Bean을 생성한다.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration
@Profile("dev")
public class ApplicationConfigDev {
  @Bean
  public ServerInfo serverInfo() {
    ServerInfo info = new ServerInfo();
    info.setIpNum("localhost");
    info.setPortNum("8181");
    return info;
  }
}
```

* ApplicationConfigRun.java

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration
@Profile("run")
public class ApplicationConfigRun {
  @Bean
  public ServerInfo serverInfo() {
    ServerInfo info = new ServerInfo();
    info.setIpNum("213.186.229.29");
    info.setPortNum("80");
    return info;
  }
}
```

* ServerInfo.java

```java
public class ServerInfo {
  private String ipNum;
  private String portNum;

  public String getIpNum() {
    return ipNum;
  }

  public void setIpNum(String ipNum) {
    this.ipNum = ipNum;
  }

  public String getPortNum() {
    return portNum;
  }

  public void setPortNum(String portNum) {
    this.portNum = portNum;
  }
}
```

* MainClass.java
  * AnnotationConfigApplicationContext를 만들고, Environment 객체를 가져와서 ActiveProfiles설정을 해준다.
  * 해당 Profiles에 해당하는 config파일로 부터 Bean을 가져와서 사용한다.

```java
import java.util.Scanner;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.GenericXmlApplicationContext;

public class MainClass {
  public static void main(String[] args) {
    String config = null;
    Scanner scanner = new Scanner(System.in);
    String str = scanner.next();
    if (str.equals("dev")) {
      config = "dev";
    } else if (str.equals("run")) {
      config = "run";
    }

    scanner.close();

    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.getEnvironment().setActiveProfiles(config);
    ctx.register(ApplicationConfigDev.class, ApplicationConfigRun.class);
    ctx.refresh();

    ServerInfo info = ctx.getBean("serverInfo", ServerInfo.class);
    System.out.println("ip : " + info.getIpNum());
    System.out.println("port : " + info.getPortNum());
    
    ctx.close();
  }
}
```



## 파일(Profile) 속성을 이용한 설