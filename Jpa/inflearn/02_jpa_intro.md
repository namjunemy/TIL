# JPA Intro

## JPA

* Java Persistence API
* 자바 진영의 ORM 기술 표준

## ORM

* Object-relational mapping(객체 관계 매핑)
  * 객체와 RDB를 어떻게 매핑할까?
* 객체는 객체대로 설계
* 관계형 데이터베이스는 관계형 데이터베이스대로 설계
* ORM 프레임워크가 중간에서 매핑
* 대중적인 언어에는 대부분 ORM 기술이 존재

## JPA의 동작 지점

* **JPA는 애플리케이션과 JDBC 사이에서 동작**한다. JDBC API 와 DB 사이에 있기 때문에 가운데서 대신 무언가를 해준다.

* 개발자는 직접 JDBC API를 쓰지 않고, JPA를 통해서 사용한다.

  ![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/02_jpa_working_point.png?raw=true)

### 저장

![](https://github.com/namjunemy/TIL/blob/master/Jpa/tacademy/img/01_extends.PNG?raw=true)

* 위의 상속관계에서 Album 객체를 저장한다고 생각해보자.

* INSERT 쿼리가 두번 만들어져서 날라간다.

  * INSERT INTO ITEM ...
  * INSERT INTO ALBUM ...

* JPA persistent 객체에 Album 객체 저장하면. 알아서 INSERT 쿼리 두개 만들어서 넣는다.

* 단순하게 INSERT 쿼리 두벌 만들어서 DB에 넣는게 아니라 **패러다임의 불일치 자체를 해결**한다.

    * 뒤에서 자세히 설명

* 알아서 두개 클래스에 다 넣어준다.

* 예)

  ![](https://github.com/namjunemy/TIL/blob/master/Jpa/tacademy/img/02_jpa_save.PNG?raw=true)

### 조회

* JPA를 통해서 MEMBER 객체를 조회하게 되면

* JPA가 매핑 정보를 바탕으로 적절한 SELECT 쿼리를 만들어서

* JDBC API를 통해 DB에 요청하고

* 결과를 받아와서 객체에 매핑해준다.

* 그래서 코드 한줄로 조회가 가능하다

* 여기서도 가장 중요한것은 패러다임의 불일치를 해결해 준다는 것이다.

* 예)

  ![](https://github.com/namjunemy/TIL/blob/master/Jpa/tacademy/img/03_jpa_find.PNG?raw=true)

## JPA 소개

* 예전에 상용 WAS를 사용하던 시절 WAS 1대당 수천만원을 호가했고, 두대정도 깔고 클러스터링 하면 5-6천만원씩 주고 서버를 구성했다. 거기에 EJB라는 기능이 들어있었는데, EJB의 엔티티 빈(자바 표준)이라는 JPA의 초창기 버전이 있었다.
* 이 엔티티 빈의 문제가 너무 많았다. 너무 코드가 지저분하고, API도 너무 복잡하고, 성능도 나빴다. EJB가 너무 문제가 많아서 개빈 킹이 오픈소스 ORM 프레임워크 하이버네이트를 출시했고
* EJB 엔티티 빈의 사용자들이 다 넘어가면서 하이버네이트가 득세를 한다.
* 그 후에 자바진영에서 자바 표준 JPA를 만들었는데, 개빈 킹을 데려와서 만들었다.
* JPA는 사실상 인터페이스만 있다. 구현은 각 기업이 하는거고, 그 중에서 오픈소스인 하이버네이트 같은 구현체들이 존재하는 것이다.
* JPA가 표준화를 거치면서 수 많은 커뮤니티의 용어 등 애매한 것들을 정리하면서 표준을 만들었다.
* **JPA는 표준 명세**이며
  * **인터페이스의 모음**이다
  * JPA 2.1 표준 명세를 구현한 3가지 구현체
    * **Hibernate**(8,90% 이상 사용), EclipseLink, DataNubleus
* 이 시기에 EJB에 대한 회의감을 느낀 사람이 한명 더 있었는데, Spring 프레임워크를 만든 로드 존슨이다. Spring과 하이버네이트가 득세를 하면서 해외에서는 거의 두가지가 기술 표준이 된다.
* 국내에서는 아쉽게도 Spring을 쓰면서 iBatis/myBatis가 득세를 하게 된다.  5-6년전에 국내에만 90% 사용률을 보였고, 해외해서는 JPA 사용률이 90%였다.

## JPA를 왜 사용해야 하는가?

* SQL 중심적인 개발에서 객체 중심으로 개발

### 생산성

* JPA와 CRUD
  * 저장 
    * jpa.persist(member)
  * 조회
    * Member member = jpa.find(memberId)
  * 수정
    * member.setName("변경할 이름")
    * 트랜잭션이 끝나는 시점에 변경된 내용 찾아서 DB에 넣어준다. 뒤에서 보자.
  * 삭제
    * jpa.remove(member)

### 유지보수

* 기존에는 필드 변경시 모든 SQL을 수정한다. 개발자가 직접 insert, select, update에 모두
* 하지만, JPA는 필드만 추가하면 된다. SQL은 JPA가 처리한다.

### 패러다임의 불일치 해결

* **JPA와 상속**

  ![](https://github.com/namjunemy/TIL/blob/master/Jpa/tacademy/img/01_extends.PNG?raw=true)

  * 저장

    * 개발자가 할 일
      * `jpa.persist(album);`
    * 나머진 JPA가 처리
      * `INSERT INTO ITEM ...`
      * `INSERT INTO ALBUM ...`

  * 조회 

    * 개발자가 할 일

      * `Album album = jpa.find(Album.class, albumId);`

    * 나머진 JPA가 처리

      ```sql
      SELECT I.*, A.*
        FROM ITEM I
        JOIN ALBUM A ON I.ITEM_ID = A.ITEM_ID
      ```

* **JPA와 연관관계**

  * 연관관계 저장

    ```java
    member.setTeam(team);
    jpa.persist(member);
    ```

* **JPA와 객체 그래프 탐색**

  ```java
  Member member = jpa.find(Member.class, memberId);
  Team team = member.getTeam();  //위에서 연관관계 저장해서 팀 넣었다. 탐색도 가능하다.
  ```

  * 뭔가 이상하지 않은가?
  * member만 가져왔는데 어떻게 team이 조회가 되지? NPE 날 것 같은데?
  * 이렇게 예측할 수 있다. JPA가 멤버랑 팀이랑 한방쿼리로 조인해서 가져오나보다.
  * 근데 테이블 50개 연관되어있다. 50개 다 조인해서 가져올까?
  * 그렇지 않다. 아주 우아한 방법으로 동작을 한다.
  * 먼저 답을 말하면 getTeam()은 가능하다. 가져올 수 있다.
  * 방법은 이렇다. member.getTeam()할 때, 정확히 말하면 가져온 team을 사용하는 시점에 team이 가져와져 있지 않으면 그때 가져온다. 팀만 조회하는 쿼리를 날려서. 이것을  **LazyLoding** 이라고 표현한다.
  * 쿼리 두번 나가잖아요! 라고 할 수 있는데, 이게 싫다면 설정을 통해서 한번에 가져오는 쿼리를 날릴 수도 있다.
  * 이렇기 때문에 **신뢰할 수 있는 엔티티 계층이 생기게 된다.** JPA를 통해서 객체를 가져왔다면, member.getTeam(), member.getOrder().getDelevery()를 이용해 자유롭게 객체 그래프를 탐색할 수 있다.

* **JPA와 비교하기**

  * JPA는 **동일 트랜잭션에서 조회한 엔티티는 같음을 보장**한다.

    * 강의 진행하면서 어떻게 같은 엔티티를 보장하는지 알게 된다.
    
    ```java
    String memberId = "100";
    Member member1 = jpa.find(memberId); //SQL
    Member member2 = jpa.find(memberId); //캐시에서 조회(아래에서 설명)
    
    member1 == member2; //같다.
    ```
    
  * JPA를 사용하는 코드를 보면 자바 컬렉션을 사용하는 느낌을 받을 수 있는데

  * 그게 JPA의 컨셉이다.

  * 나머지는 JPA가 해주겠다는 의미이다.

### JPA의 성능 최적화 기능

* JPA가 JDBC API와 DB사이에 존재하므로 가운데에 계층이 추가되서 느리지 않을까? 라고 생각할 수 있지만, 반대로 중간에 계층이 있으면 할 수 있는게 많다.(쉽게 말하자면 모아서 쿼리를 쏘는 버퍼링과 읽을 때 사용 가능한 캐싱을 할 수 있다.)

  * **1차 캐시와 동일성(identity) 보장**

    * 같은 트랜잭션 안에서는 같은 엔티티를 반환한다. 약간의 조회 성능이 향상된다.
  * 바로 위의 JPA와 비교하기의 코드에서 jpa.find()가 두번 일어나는데, JPA는 **같은 트랜잭션에서** 두번째 find는 1차 캐시에서 가져온다.
    * 결과적으로 SQL이 1번만 실행된다.
  * 이것은 우리가 아는 캐시가 아니다. 클라이언트 요청이 한번 오면, 내부 원사이클을 수행하고 빠져나가게 되는데 그 요청 한번 이 시작되고 끝날때까지의 트랜잭션 사이에서의 동일성을 보장한다. 1차 캐시의 매커니즘을 이해하는 것이 중요하다.
  
* **트랜잭션을 지원하는 쓰기 지연(transactional write-behind)**
  
  * 트랜잭션을 커밋할 때까지 INSERT SQL을 모음
  
  * **JDBC BATCH SQL** 기능을 사용해서 한번에 SQL 전송
  
      * 모으지 않으면 네트워크 3번 타고, 쿼리를 3번 날린다.
  
      ```java
      transcation.begin(); //트랜잭션 시작
      
      em.persist(memberA);
      em.persist(memberA);
      em.persist(memberA);
      //여기까지 insert SQL을 DB에 보내지 않는다.
      
    //커밋하는 순간 DB에 SQL 모아서 보낸다.
      transction.commit();
    ```
  
* 지연 로딩(Lazy Loading)

    - 지연로딩 : **객체가 실제 사용될 때** 로딩

      ```java
      Member member = memberDAO.find(memberId);  // SELECT * FROM MEMBER
      Team team = member.getTeam();
      String teamName = team.getName();  //SELECT * FROM TEAM
      ```

    - 즉시로딩: JOIN SQL로 한번에 연관된 객체까지 미리 조회

      ```java
      Member member = memberDAO.find(memberId);  // SELECT ... FROM MEMBER JOIN TEAM ...
      Team team = member.getTeam();
      String teamName = team.getName();
      ```

### ORM은 객체와 RDB 두 기둥위에 있는 기술이다

* 객체도 잘 다뤄야 하고, RDB도 정말 잘 다뤄야 한다. 실제 현업에서 90% 이상 장애는 DB에서 난다.
* JPA로 개발하더라도 쿼리가 눈에 다 보여야 한다. 실제로 JPA는 정말 복잡한 쿼리가 아닌 심플한 쿼리들이 나간다.

### Reference

- [자바 ORM 표준 JPA 프로그래밍](https://www.inflearn.com/course/ORM-JPA-Basic)