# DI(Dependency Injection)를 이용한 빈 의존성 관리

> '자바 웹 개발 워크북 - 엄진영'을 참고하여 학습한 내용입니다.

MVC아키텍처에서 Controller가 작업을 수행하려면 데이터베이스로 부터 정보를 가져다줄 DAO가 필요하다. 이렇게 특정작업을 수행할 때 사용하는 객체를 **의존 객체** 라고 하고, 이런 관계를 **의존 관계(Dependency)** 라고 한다.



### 의존 객체 필요시 즉시 생성

의존 객체를 관리하는 방법은 두가지가 있다. 먼저 고전적인 방법은 의존 객체를 사용하는 쪽에서 직접 그 객체를 생성하고 관리하는 것이다. 다음의 예를 살펴보자.

```java
public void doGet(HttpServletRequest request, HttpServletResponse response)
  throws ServletException, IOException {
    try {
      ServletContext sc = this.getServletContext();
      Connection conn = (Connection) sc.getAttribute("conn");
      
      //회원 목록을 가져오기 위해 직접 DAO 객체 생성
      MemberDao memberdao = new MemberDao();
      memberDao.setConnection(conn);
      
      request.setAttrubute("members", memberDao.selectList());
      ...
    }
  	...
}
```

위의 발식은 회원 목록 데이터를 가져오기 위해 직접 MemberDao 객체를 생성하고 있다. 이 방식의 문제점은 doGet()이 호출될 때마다 MemberDao 객체를 생성하기 때문에 비효율 적이다.



### 의존 객체를 미리 생성해 두고 필요할 때 사용

앞의 방법을 조금 개선한 것이 사용할 객체를 미리 생성해 두고 필요할 때마다 꺼내 쓰는 방식이다. 웹 어플리케이션이 시작될 때 MemberDao 객체를 미리 생성하여 ServletContext에 보관해두고, 필요할 때마다 꺼내 쓴다.

```java
publid void doGet(HttpServletRequest request, HttpServletResponse response)
  throws ServletException, IOException {
    try {
      ServletContext sc = this.getServletContext();
      MemberDao memberDao = (MemberDao)sc.getAttribute("memberDao");
      memberDao.setConnection(conn);
      
      request.setAttribute("members", memberDao.selectList());
      ...
    }
  	...
}
```



### 의존 객체와의 결합도 증가에 따른 문제

위에서 처럼 의존 객체를 직접 생성하거나 보관소에서 꺼내는 방식으로 관리하다 보니 문제가 발생한다.

* 코드의 잦은 변경

  의존 객체를 사용하는 쪽과 의존 객체(또는 보관소) 사이의 결합도가 높아져서 의존 객체나 보관소에 변경이 발생하면 바로 영향을 받는다. 예를 들어 의존 객체의 기본 생성자가 public에서 private으로 바뀌면 의존 객체를 생성하는 모든 코드를 찾아 변경해 주어야 한다. 보관소도 마찬가지이다.

* 대체가 어렵다

  의존 객체를 다른 객체로 대체하기가 어협다. 현재 Controller가 사용하는 DAO는 MySQL을 사용하도록 작성되었다고 가정하자. 만약 Oracle 데이터베이스를 사용해야 한다면, 일부 SQL 문에 맞게끔 변경해야한다.

  다른 방법으로는 각 데이터베이스 별로 DAO를 준비하는 것인데, 이것도 데이터베이스가 바뀔 때마다 DAO를 사용하는 코드를 변경해주어야 하는 문제점이 있다.

  ​



### 의존 객체를 외부에서 주입

책에서 인간의 실제 생활을 비교해서 설명했다. 규모가 작은 원시 사회에서는 사람들이 직접 필요한 도구들을 만들어 썻다. 농사도 직접 짓고, 집도 직접 만들고, 사냥도 직접했다. 그러나 사회 규모가 커짐에 따라 분업화가 진행되어 일부는 개인이 직접 만들지만, 대부분은 남이 만든 것을 이용하게 되었다.  당장 우리가 사용하는 컴퓨터나 노트북도 그러하다.

프로그래밍도 현실 세계와 똑같다. 초창기 객체지향 프로그래밍에서는 의존 객체를 직접 생성했다. 그러다가 위에서 언급한 문제를 해결하기 위해 의존 객체를 외부에서 주입받는 방식(Dependency Injection)으로 바뀌게 된다.

따라서, 의존 객체를 전문으로 관리하는 '빈 컨테이너(Java Beans Container)'가 등장하게 됐다. 빈 컨테이너는 객체가 실행되기 전에 그 객체가 필요로 하는 의존 객체를 주입해주는 역할을 수행한다. 이런 방식으로 의존 객체를 관리하는 것을 '의존성 주입( DI; Dependency Injection)'이라 한다. 좀 더 일반적인 말로 '역제어(IoC; Inversion of Control)'라고 부른다. 즉 역제어(IoC)의 한 예가 의존성 주입(DI)이다.



### DAO와 DataSource

지금까지 사용했던 DAO에 DI 기법이 이미 적용되어 있다. Dao가 작업을 수행하려면 데이터베이스와 연결을 수행하는 DataSource가 필요하다. 그런데 이 DataSource 객체를 Dao에서 직접 생성하는 것이 아니라 외부에서 주입 받는다. 다음은 DAO에 DataSource를 주입하는 코드이다.

> .../listeners/ContextLoaderListener.java의 일부분

```java
...
  
public void contextInitialized(ServletContextEvent event) {
    try {
      ServletContext sc = event.getServletContext();
      
      InitialContext initialContext = new InitialContext();
      DataSource ds = (DataSource)initialContext.lookup("java:comp/env/jdbc/studydb");
      
      MemberDao memberDao = new MemberDao();
      memberDao.setDataSource(ds);
      ...
    }
    ...
}

...
```

ContextLoaderListener의 contextInitialized()는 웹 애플리케이션이 시작될 때 호출되는 메서드이다. 이 메서드를 살펴보면 setDataSource()를 호출하는 부분이 있다. MemberDao가 사용할 의존 객체인 'DataSource'를 주입하고 있다.

> memberDao.setDataSource(ds);



### Controller에 Dao주입

Controller가 작업을 수행하려면 Dao가 필요하다. 실제로 MemberListController에 DI를 적용해 본다.

```java
public class MemberListController implements Conteoller {
  MemberDao memberDao;
  
  public MemberListController setMemberDao(MemberDao memberDao) {
    this.memberDao = memberDao;
    return this;
  }
  
  @Override
  public String execute(Map<String, Object> model) throws Exception {
    MemberDao memberDao = (MemberDao)model.get("memberDao");
    
    model.put("members", memberDao.selectList());
    
    return "/member/MemberList.jsp";
  }
}
```

* 의존 객체 주입을 위한 인스턴스 변수와 셋터 메서드

  Dao에서 DataSource 객체를 주입 받고자 인스턴스 변수와 셋터 메서드를 선언했듯이, MemberListController에도 MemberDao를 주입 받기 위한 인스턴스 변수와 셋터 메서드를 추가했다.

  ```java
    public MemberListController setMemberDao(MemberDao memberDao) {
      this.memberDao = memberDao;
      return this;
    }
  ```

  셋터 메서드를 좀 더 쉽게 사용하기 위해 Member 클래스에서처럼 리턴 타입이 그 자신의 인스턴스 값을 리턴하게 했다.

* 의존 객체를 스스로 준비하지 않는다.

  외부에서 MemberDao객체를 주입해 줄 것이기 때문에 이제 더 이상 Map 객체에서 MemberDao를 꺼낼 필요가 없다. 따라서 MemberListController에서 해당 코드를 제거한다.

  > MemberDao memberDao = (MemberDao)model.get("memberDao");