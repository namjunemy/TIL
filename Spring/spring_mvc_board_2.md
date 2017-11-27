# 17. 스프링MVC 게시판 2

## 17-1. 패키지, 인터페이스, 클래스 제작

프로젝트 설계에 따른 전체적인 패키지, 인터페이스, 기본 클래스들을 만든다.

* 컨트롤러를 만들어서 @RequestMapping을 통해 요청을 처리하도록 한다.
* 실제로 작업내용이 들어있는 Command 클래스들을 만든다.
* DB에 접근하기 위한 DAO클래스와 실제 값들을 담기 위한 DTO클래스를 만든다
* 학습에서는 아래의 프로젝트 구조대로 각 클래스파일을 만들어 본다.

![](https://github.com/namjunemy/TIL/blob/master/Spring/img/spring_mvc_2.png?raw=true)



## 17-2. Controller 제작

클라이언트의 요청에 따른 전체적인 작업을 지휘하는 Controller를 만들어 본다.

JSP의 MVC Model2 방식을 기억하면서 작업을 진행한다.

* 먼저 Client 요청이 들어온다.
  * JSP & Servlet에서는 클라이언트의 요청이 들어오면 요청에 따라서 Servlet Mapping에 따라서 Controller를 만들었다.
* Spring MVC에서는 Client의 요청이 들어오면, Dispatcher에서 Auto Scan을 통해 요청을 처리할 Controller에게 요청을 전달 한다.
  * 이 작업들은 우리가 Spring MVC 프로젝트를 만드는 순간 자동으로 생성된다.(web.xml파일, servlet-context.xml파일)
* 17-1에서 만들어 놓은 Command 클래스로 연결할 컨트롤러 메소드를 정의한다.
  * 게시물 리스트 출력 : list()
  * 게시물 등록 내용 입력 화면 출력 : writeView()
  * 게시물 등록 : write()
  * 게시물 상세 조회 : contentView()
  * 게시물 수정 : modify()
  * 댓글 등록 내용 입력 화면 출력 : replyView()
  * 댓글 등록 : reply()
  * 게시물 삭제 : delete()

```java
package com.javalec.spring_project_board_controller;

import javax.servlet.http.HttpServletRequest;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.javalec.spring_project_board_command.BCommand;
import com.javalec.spring_project_board_command.BContentCommand;
import com.javalec.spring_project_board_command.BDeleteCommand;
import com.javalec.spring_project_board_command.BListCommand;
import com.javalec.spring_project_board_command.BModifyCommand;
import com.javalec.spring_project_board_command.BReplyCommand;
import com.javalec.spring_project_board_command.BReplyViewCommand;
import com.javalec.spring_project_board_command.BWriteCommand;

@Controller
public class BController {
  BCommand command;

  @RequestMapping("/list")
  public String list(Model model) {
    System.out.println("list()");

    command = new BListCommand();
    command.execute(model);

    return "list";
  }

  @RequestMapping("/writeView")
  public String writeView(Model model) {
    System.out.println("writeView()");

    return "writeView";
  }

  @RequestMapping("/write")
  public String write(HttpServletRequest request, Model model) {
    System.out.println("write()");

    model.addAttribute("request", request);
    command = new BWriteCommand();
    command.execute(model);

    return "redirect:list";
  }

  @RequestMapping("/contentView")
  public String contentView(HttpServletRequest request, Model model) {
    System.out.println("contentView()");

    model.addAttribute("request", request);
    command = new BContentCommand();
    command.execute(model);

    return "contentView";
  }

  @RequestMapping(method=RequestMethod.POST, value = "/modify")
  public String modify(HttpServletRequest request, Model model) {
    System.out.println("modify()");
    
    model.addAttribute("request", request);
    command = new BModifyCommand();
    command.execute(model);
    
    return "redirect:list";
  }
  
  @RequestMapping("/replyView")
  public String replyView(HttpServletRequest request, Model model) {
    System.out.println("replyView()");
    
    model.addAttribute("request", request);
    command = new BReplyViewCommand();
    command.execute(model);    
    
    return "replyView";
  }
  
  @RequestMapping("/reply")
  public String reply(HttpServletRequest request, Model model) {
    System.out.println("reply()");
    
    model.addAttribute("request", request);
    command = new BReplyCommand();
    command.execute(model);    
    
    return "redirect:list";
  }
  
  @RequestMapping("/delete")
  public String delete(HttpServletRequest request, Model model) {
    System.out.println("delete");
    
    model.addAttribute("request", request);
    command = new BDeleteCommand();
    command.execute(model);
    
    return "redirect:delete";
  }
}
```



## 17-3. 리스트 페이지 만들기

사용자의 요청에 의해 게시판 리스트 페이지를 보기까지의 과정은 다음과 같다.

* 사용자의 요청을 Controller에서 @RequestMapping에 정의 된 해당 url을 처리한다.
* 이 메소드 안에서, 실제 작업이 이루어질 Command 객체를 만들고 작업을 수행한다.
* Command객체에서는 내부적으로 DAO를 만들어서 DB에 접근하여 조회를 원하는 데이터를 가져온 뒤,
* model 객체에 추가하여 JSP페이지에서 사용 가능하게 한다.
* JSP에서는 model객체를 참조하여 화면을 구성한다.

![](https://github.com/namjunemy/TIL/blob/master/Spring/img/spring_mvc_3.png?raw=true)









