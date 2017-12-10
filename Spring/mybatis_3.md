# 30. Mybatis-3

## 30-1 Mybatis를 이용한 글 작성 및 삭제

[실습코드 repo](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_28_1_ex1_springex)

* insert
  * \#{param1} : database 컬럼 타입에 맞게 변환을 해서 넣는다.
  * ${param1} : 변환 없이 그냥 들어오는 타입대로 넣는다.  
* delete
* IDao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.javalec.mybatis.dao.IDao">
  <select id="listDao" resultType="com.javalec.mybatis.dto.ContentDto">
    SELECT * FROM BOARD ORDER BY
    MID DESC
  </select>

  <insert id="writeDao">
    INSERT INTO BOARD (mWriter, mContent) VALUES (#{param1}, #{param2})
  </insert>
  
  <delete id="deleteDao">
    DELETE FROM BOARD WHERE mId = #{param1}
  </delete>

</mapper>
```

* HomeController.java

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

import com.javalec.mybatis.dao.IDao;

@Controller
public class HomeController {
  private static final Logger logger = LoggerFactory.getLogger(HomeController.class);

  @Autowired
  private SqlSession sqlSession;

  @RequestMapping(value = "/", method = RequestMethod.GET)
  public String home(Locale locale, Model model) {
    logger.info("Welcome home! The client locale is {}.", locale);

    Date date = new Date();
    DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG, locale);

    String formattedDate = dateFormat.format(date);

    model.addAttribute("serverTime", formattedDate);

    return "home";
  }

  @RequestMapping("/list")
  public String list(Model model) {
    IDao dao = sqlSession.getMapper(IDao.class);
    model.addAttribute("list", dao.listDao());

    return "/list";
  }

  @RequestMapping("writeForm")
  public String writeForm() {
    return "/writeForm";
  }

  @RequestMapping("write")
  public String write(HttpServletRequest request, Model model) {
    IDao dao = sqlSession.getMapper(IDao.class);
    dao.writeDao(request.getParameter("mWriter"), request.getParameter("mContent"));

    return "redirect:list";
  }

  @RequestMapping("view")
  public String view() {
    return "view";
  }

  @RequestMapping("/delete")
  public String delete(HttpServletRequest request, Model model) {
    IDao dao = sqlSession.getMapper(IDao.class);
    dao.deleteDao(request.getParameter("mId"));

    return "redirect:list";
  }
}
```



## 30-2 소스 정리 및 마무리 글

* Spring 가장 중요한 것 injection. spring IoC 컨테이너에 Bean을 만들어 놓고, 생성하고 조립하는 과정.
* Spring 에서 가장 어려운 부분은 설정하는 부분. 많은 기능들을 사용하기 위해 설정하는 부분들과 그 기능들의 버전 업데이트 주기가 다름에 따라 오는 어려움 등이 있다.
* 많은 경험과 디버깅을 통해서 어려움을 이겨나가는 노하우를 얻는 것이 필요하다.

