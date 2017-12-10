# 29.Mybatis-2

저번 시간에 구성했던 기본 프로젝트 설정을 바탕으로 mybatis를 적용해 본다.

## 29-1. Mybatis를 이용한 리스트 출력

Dao를 xml로 뽑아내서 사용할 수 있다.

* spring-context.xml에 mybatis Bean 관련 설정 추가

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/mvc"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:beans="http://www.springframework.org/schema/beans"
             xmlns:context="http://www.springframework.org/schema/context"
             xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

  <!-- DispatcherServlet Context: defines this servlet's request-processing 
    infrastructure -->

  <!-- Enables the Spring MVC @Controller programming model -->
  <annotation-driven/>

  <!-- Handles HTTP GET requests for /resources/** by efficiently serving 
    up static resources in the ${webappRoot}/resources directory -->
  <resources mapping="/resources/**" location="/resources/"/>

  <!-- Resolves views selected for rendering by @Controllers to .jsp resources 
    in the /WEB-INF/views directory -->
  <beans:bean
      class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <beans:property name="prefix" value="/WEB-INF/views/"/>
    <beans:property name="suffix" value=".jsp"/>
  </beans:bean>

  <context:component-scan base-package="com.javalec.ex"/>


  <beans:bean name="dataSource"
              class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <beans:property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <beans:property name="url"
                    value="jdbc:mysql://localhost:3306/springmvcdb"/>
    <beans:property name="username" value="ID"/>
    <beans:property name="password" value="PW"/>
  </beans:bean>

  <beans:bean name="template"
              class="org.springframework.jdbc.core.JdbcTemplate">
    <beans:property name="dataSource" ref="dataSource"/>
  </beans:bean>

  <beans:bean name="dao" class="com.javalec.mybatis.dao.ContentDao">
    <beans:property name="template" ref="template"/>
  </beans:bean>
  
  <beans:bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <beans:property name="dataSource" ref="dataSource"/>
    <beans:property name="mapperLocations" value="classpath:com/javalec/mybatis/dao/mapper/*.xml"/>
  </beans:bean>
  
  <beans:bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
    <beans:constructor-arg index="0" ref="sqlSessionFactory"/>
  </beans:bean>
  
</beans:beans>
```

* IDao.xml 파일 추가
  * 인터페이스를 namespace로 사용하기 위한 mapper 추가
  * select, insert, delete 태그 추가
    * 인터페이스에 정의 된 메소드명, resultType 정의

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.javalec.mybatis.dao.IDao">
  <select id="listDao" resultType="com.javalec.mybatis.dto.ContentDto">
    SELECT * FROM BOARD ORDER BY MID DESC
  </select>
  
</mapper>
```

* Dao 객체 사용처
  * @Autowired
    * setter 없이 @Autowired를 통해 sqlSession을 사용할 수 있다.
  * sqlSession.getMapper(Dao.class);

```java
package com.javalec.ex;

import java.text.DateFormat;
import java.util.Date;
import java.util.Locale;

import javax.servlet.http.HttpServletRequest;

import org.apache.ibatis.session.SqlSession;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.javalec.mybatis.dao.ContentDao;
import com.javalec.mybatis.dao.IDao;

@Controller
public class HomeController {
  private static final Logger logger = LoggerFactory.getLogger(HomeController.class);

  ContentDao dao;

  @Autowired
  private SqlSession sqlSession;

  @Autowired
  public void setDao(ContentDao dao) {
    this.dao = dao;
  }

  @RequestMapping("/list")
  public String list(Model model) {
    IDao dao = sqlSession.getMapper(IDao.class);
    model.addAttribute("list", dao.listDao());

    return "/list";
  }
  
  ...
}
```

