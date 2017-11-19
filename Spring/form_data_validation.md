# 15. Form 데이터 값 검증

클라이언트에서 전송한 데이터를 서버에서 처리한 후 응답하는 과정에서, Form데이터의 제약조건, 필수조건 등을 자바스크립트를 통해서 해결할 수 있다. 이번에 학습 할 것은 클라이언트에서 검증하는 것이 아닌 서버 쪽에서 검증하는 방법을 학습한다. 

## 15-1. Validator 인터페이스를 이용한 검증

폼에서 전달되는 데이터를 커맨드 객체에 담아 컨트롤 객체에 전달 한다고 했다. 이때, 커맨드 객체의 유효성 검사를 할 수 있다. 지금 하는 Validator 인터페이스를 이용하는 방법은 서버에서 Validation을 검사하는 방법이다.

* 흐름은 다음과 같다.

  * 클라이언트에서 Form에 데이터를 입력해서 server로 전송한다.
  * 이때, 데이터가 많을 경우 커맨드 객체에 담아서 넘어온다.
  * 컨트롤러에서 Validator 객체를 만들어서 유효성 검증을 먼저 한다. 검증에 이상이 없으면,
  * 그 다음에 뷰쪽으로 응답한다.
  * 검증에 통과하지 못하면, 다시 클라이언트로 에러를 돌려보낸다.

* 컨트롤러 코드 예시

  * 요청을 처리하는 메소드의 인자로 Student 데이터 객체와, BindingResult 객체를 넘긴다. 
  * BindingResult객체는 Validation 결과를 Binding해서 검증을 확인하는데 사용되는 객체이다.
  * Validator객체의 인스턴스를 만들고, 인스턴스의 validate메소드를 통해 student와 BindingResult객체를 이용하여 검증한다.
  * BindingResult의 hasErrors() 메소드 리턴 값이 있으면 검증결과 에러가 있는 것이므로, 에러페이지에 해당하는 페이지 이름을 할당한다.
  * 검증에 통과할 경우 정상 페이지를 return 한다.

  ```java
  ...
  @RequestMapping("/student/create")
  public String studentCreate(@ModelAttribute("student") Student student, BindingResult result) {
    String page = "createDonePage";
    
    StudentValidator validator = new StudentValidator();
    validator.validate(student, result);
    if(result.hasErrors()) {
      page = "createPage";
    }
    
    return page;
  }
  ...
  ```

* Validator의 validate() 메소드

  * 기본적으로 Validator인터페이스의 구현객체는 두가지 메소드를 오버라이딩 해야한다.
  * supports메소드는 검증할 객체의 클래스 타입 정보를 명시해주는 역할을 한다.
  * validate() 메소드는 검증 항목을 만족하지 않으면 errors 객체에 값을 넣어준다.

  ```java
  public class StudentValidator implements Validator {

    @Override
    public boolean supports(Class<?> arg0) {
      return Student.class.isAssignableFrom(arg0); // 검증할 객체의 클래스 타입 정보
    }

    @Override
    public void validate(Object obj, Errors errors) {
      System.out.println("validate()");
      Student student = (Student) obj;

      String studentName = student.getName();
      if (studentName == null || studentName.trim().isEmpty()) {
        System.out.println("studentName is null or empty");
        errors.rejectValue("name", "trouble");
      }

      int studentId = student.getId();
      if (studentId == 0) {
        System.out.println("studentId is 0");
        errors.rejectValue("id", "trouble");
      }
    }
  }
  ```

#### 실습코드

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_15_1_ex1_springex)

  

## 15-2. ValidationUtils 클래스

데이터 검증을 위해서 Validator 인터페이스의 validate() 메소드를 사용했다. ValidationUtils 클래스는 validate() 메소드를 좀 더 편리하게 사용할 수 있도록 고안된 클래스이다.

* 기존의 validate() 메소드의 일부분

```java
...
    String studentName = student.getName();
    if (studentName == null || studentName.trim().isEmpty()) {
      System.out.println("studentName is null or empty");
      errors.rejectValue("name", "trouble");
    }
...
```

* 위의 부분을 다음과 같이 간단하게 변경하여 사용할 수 있다.

```java
ValidationUtils.rejectIfEmptyOrWhitespace(errors, "name", "trouble");
```

#### 실습코드

* 실습 코드 repo : [저장소](https://github.com/namjunemy/spring_for_junior_developer/tree/master/spring_15_2_ex1_springex)

  

## 15-3. @Valid와 @InitBinder

데이터 검증을 하기 위해서 Validator 인터페이스를 구현한 클래스를 만들고, validate()메소드를 직접 호출하여 사용했다. 이번에는 직접 호출하지 않고, 스프링 프레임워크에서 호출하는 방법에 대해서 학습한다.

* 먼저 아래의 의존을 추가한다.

  ```xml
  <dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-valudator</artifactId>
    <version>4.2.0.Final</version>
  </dependency>
  ```

* 의존을 추가하고, @InitBinder와 @Valid 어노테이션을 추가함으로써, 실제 Validator객체를 만들고 validate() 메소드를 호출했던 부분을 생략할 수 있다.

  ```java
  @RequestMapping("/student/create")
  public String studentCreate(@ModelAttribute("student") Student student, BindingResult result) {
    String page = "createDonePage";

    StudentValidator validator = new StudentValidator();
    validator.validate(student, result);
    if (result.hasErrors()) {
      page = "createPage";
    }

    return page;
  }
  ```

  * 변경 후 코드
    * @InitBinder 어노테이션을 이용하여 Validator를 set해주고, @Valid 어노테이션을 통해서 spring프레임워크에서 호출하여 검증한다.

  ```java
  @RequestMapping("/student/create")
  public String studentCreate(@ModelAttribute("student") @Valid Student student, BindingResult result) {
    String page = "createDonePage";

    // StudentValidator validator = new StudentValidator();
    // validator.validate(student, result);
    if (result.hasErrors()) {
      page = "createPage";
    }

    return page;
  }

  @InitBinder
  protected void initBinder(WebDataBinder binder) {
    binder.setValidator(new StudentValidator());
  }
  ```

  ​