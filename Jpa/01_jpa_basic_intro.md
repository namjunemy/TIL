# JPA Intro

1. SQL  중심적인 개발의 문제점
2. JPA 소개

## 1. SQL 중심적인 개발의 문제점

현재 데이터베이스 세계의 헤게모니를 관계형 DB가 가지고 있다(Oracle, MySQL, ...). 객체를 관계형 DB에 관리하고 있다는 이야기이다. 관계형 DB를 사용하려면 SQL을 짜야한다. 계속. SQL 중심적인 개발을 하게되면 아래와 같은 문제점이 있다.

### 무한 반복, 지루한 코드

* 객체 CRUD(insert, update, select, delete)

  ```java
  public class Member {
    private String memberId;
    private String name;
    ...
  }
  ```

  ```sql
  INSERT INTO MEMBER(MEMBER_ID, NAME) VALUES ...
  SELECT MEMBER_ID, NAME FROM MEMBER M
  UPDATE MEMBER SET ...
  ```

* 기획자가 tel 정보를 추가하자고 한다. 그렇다면 전개는?

  * Member 객체에 필드를 하나 추가하게되고, 모든 쿼리에 tel 정보를 추가해야 된다. **모든!** 쿼리에..

    ```java
    public class Member {
      private String memberId;
      private String name;
      private String tel;
      ...
    }
    ```

    ```sql
    INSERT INTO MEMBER(MEMBER_ID, NAME, TEL) VALUES ...
    SELECT MEMBER_ID, NAME, TEL FROM MEMBER M
    UPDATE MEMBER SET ... TEL =?
    ```

### 엔티티 신뢰 문제

* 코드를 객체 지향적으로 짰다고 가정하고, MemberDAO에서 member를 하나 꺼내왔다고 하자.
* 해당 멤버의 팀과 주문정보, 그 주문정보의 배송지 정보를 getter로 가져오려고 하는 건 자연스러운 생각이다.
* 하지만, 해당 member가 팀과 주문과 배송지 정보를 가지고 있다고 보장되지 않는 이상 이렇게 코드를 짤 수 없다.
* DAO의 코드를 까서 팀과 주문정보 주문에 대한 배송지 정보들을 쿼리로 정말 가져오는지 눈으로 확인해봐야 한다.
* 계층형 아키텍처(controller, domain, service등의 계층형 구조)의 진정한 의미의 계층 분할이 어렵다는 이야기이다. 즉, 물리적으로는 분리되어 있지만 논리적으로는 전혀 분할되어 있지 않다는 이야기 이다.
* 결과적으로 어떤 쿼리를 짜느냐에 따라 비즈니스 로직에 영향을 주기 때문에 SQL 의존적인 개발을 피하기 어렵다.

```java
class memberService {
  ...
  public void process(String id) {
    Member member = memberDAO.find(id);
    member.getTeam();									//???
    member.getOrder().getDelevery();	//???
  }
}
```

### 패러다임의 불일치

* 객체 vs 관계형 데이터베이스
  * **관계형 데이터베이스**는 철저히 '데이터를 어떤식으로 잘 저장할까'에 포커스가 맞춰져있다.
  * **객체**는 데이터 저장이 아니라 '어떻게 더 추상화하고 관리를 잘 할까'가 더 중요하다.
  * 우리는 포커스가 다른 두 가지를 억지로 맵핑해서 일을 처리해야하기 때문에 그 사이에서 많은 시간을 보내고 있다.
* 객체를 영구 보관하는 다양한 저장소
  * Object -> RDB, NoSQL, File 등 객체를 저장할 수 있는 방법이 많이 있지만,
  * 현실적인 대안은 관계형 데이터베이스이다.

* 객체를 관계형 DB에 저장하려면 객체를 SQL로 바꾼다. (개발자가, 한땀한땀)
  * 개발자가 SQL 매퍼의 일을 너무 많이 하고 있다.

### 객체와 관계형 데이터베이스의 차이

* 상속
  * 객체의 상속 관계
  * Table 슈퍼타입 서브타입 관계

* 연관관계

  * 객체는 참조를 사용

    * member.getTeam()
    * 객체의 연관관계에는 **방향성이 있다**. 멤버에서 팀을 조회할 수 있지만, 팀에서 멤버조회는 불가하다.
      * Member의 필드는 id, Team, username
      * Team의 필드는 id, name 이라고 가정.

  * 테이블은 외래 키를 사용

    * JOIN ON M.TEAM_ID = T.TEAM_ID

      * MEMBER 테이블
        * MEMBER_ID(PK)
        * TEAM_ID(FK)
        * USERNAME
      * TEAM 테이블
        * TEAM_ID(PK)
        * NAME
      * MAEMBER와 TEAM은 N:1 관계

    * 테이블의 외래키에는 **방향성이 없다**. 멤버랑 팀 조인가능, 팀과 멤버 조인가능.

  * 이 때, 위의 **테이블 설계에 맞추어 객체를 모델링**하게 되면 아래와 같이 FK를 그대로 필드로 포함하게 된다. 하지만, Member클래스에 Team 객체가 존재하는 것이 더 객체지향적이다 라고 할 수 있다. FK의 값을 넣는것 보단.

    ```java
    class Member {
      String id;        //MEMBER_ID 컬럼
      Long teamId;      //TEAM_ID FK 컬럼
      String username;  //USERNAME 컬럼
    }
    ```

    ```java
    class Team {
      Long id;
      String name;
    }
    ```

    * 이렇게 설계된 객체를 테이블에 저장 한다.

      * 테이블 설계에 맞추어 객체를 모델링해서 INSERT 쿼리를 짠다.

        ```sql
        INSERT INTO MEMBER(MEMBER_ID, TEAM_ID, USERNAME) VALUES ...
        ```

  * 그러나, **객체다운 모델링**에서는 아래와 같이 Team이라는 객체 자체를 포함하고 있어서 Member객체에서 Team을 바로 접근할 수 있다.
     ```java
     class Member {
       String id;
       Team team;        //참조로 연관관계를 맺는다
       String username;
     }
     ```

     ```java
     class Team {
       Long id;        // TEAM_ID PK 사용
       String name;    // NAME 컬럼 사용
     }
     ```

     * 테이블에 저장할 때에는 `member.getTeam().getId()`로 Id를 조회해서 넣었다.

        ```sql
        INSERT INTO MEMBER(MEMBER_ID, TEAM_ID, USERNAME) VALUES ...
        ```

     * 그러나, 조회를 하려고 하면…. 헬게이트 오픈이다. 먼저 멤버와 팀을 조인해 놓고 팀을 조회할 준비를 한다.

        ```sql
        SELECT M.*, T.*
          FROM MEMBER M
          JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
        ```

     * DB에서 조회해서 객체에 넣으려면.. 

        ```java
        public Member find(Strubg memberId) {
          // SQL 실행하고
          Member member = new Member();
          // DB에서 조회한 회원 관련 정보를 모두 입력하고
          Team team = new Team();
          // DB에서 조회한 팀 관련 정보를 모두 넣고,
          
          // 회원과 팀 관계 설정
          member.setTeam(team);
          return member;
        }
        ```

     * 하지만…. 선배들은 member와 team의 정보를 모두 가지고 있는 member_team DTO를 가지고,  위와같이 복잡하게 연관관계 매핑을 하지않고 한방 쿼리를 날리면서 작업을 한다...

* 객체 그래프 탐색

  * 객체는 자유롭게 객체 그래프를 탐색할 수 있어야 한다.

    * 예를 들면, member.getTeam(), member.getOrder(), member.getOrder().getOderItem() 할 수 있어야 된다.

  * DB에서는 멤버와 팀 멤버와 오더 서로 조회를 할 수 있다. 하지만, 서비스 로직에서는 못한다. 왜? 처음에 쿼리에 넣어놓지 않아서 가져오지 않았기 때문에.

  * 핵심은 **처음 실행하는 SQL에 따라 탐색 범위가 결정**된다는 것이다.

    ```sql
    SELECT M.*, T.*
      FROM MEMBER M
      JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
    ```

    ```java
    member.getTeam(); //OK
    
    member.getOrder(); //NULL
    ```

  * 아래와 같은 서비스 로직을 짤 때, 내가 DAO를 작성하지 않았다고 생각하면 헬게이트 오픈. memberDAO가 member를 어떻게 가져오느냐 눈으로 확인해야된다. NPE 여지가 다분하다.

      ```java
      class MemeberService {
        ...
        public void process() {
        	Member member = memberDAO.find(memberId);
          member.getTeam();  //???
          member.getOrder().getDelivery();   //???
        }
      }
      ```

  * 이런식의 해결법도 있긴 하다. 

    * 경우의 수를 다 파악해서 DAO에 메소드를 다 만들어 놓는다.
    * Member만 조회, Member와 Team 조회, Order까지 조회 등
    * 이게 맞을까?

* 비교하기

    * DB에서 조회해오면 JDBC 접근로직 타고 new로 생성하니까 당연히 다르다.

        ```java
        String memberId = "100";
        Member member1 = memberDAO.getMember(memberId);
        Member member2 = memberDAO.getMember(memberId);
        
        member1 == member2; //다르다
        ```

        ```java
        class MemberDAO {
          public Member getMember(String memberId) {
            String sql = "SELECT * FROM MEMBER WHERE MEMBER_ID = ?";
          	...
            // JDBC API, SQL 실행
            return new Member(...);
          }
        }
        ```

* 비교하기 - 자바 컬렉션에서 조회

    * 그런데, 자바 컬렉션에서 조회한다고 가정해보면 두 멤버는 같은 멤버이다.

        ```java
        String memberId = "100";
        Member member1 = list.get(memberId);
        Member member2 = list.get(memberId);
        
        member1 == member2; //같다.
        ```

* 위의 비교를 통해 둘간의 패러다임 차이를 느낄 수 있다.
* 결론적으로 객체답게 모델링 할수록 매핑 작업만 늘어나게 된다.
* 객체를 자바 컬렉션에 저장하듯이 DB에 저장할 수는 없을까?
* 이에 대한 고민에 의해 자바진영에는 JPA라는 것이 있다.

## 2.  JPA?

* Java Persistence API
* 자바 진영의 ORM 기술 표준

### ORM

* Object-relational mapping(객체 관계 매핑)
  * 객체와 RDB를 어떻게 매핑할까?
* 객체는 객체대로 설계
* 관계형 데이터베이스는 관계형 데이터베이스대로 설계
* ORM 프레임워크가 중간에서 매핑
* 대중적인 언어에는 대부분 ORM 기술이 존재

### JPA의 동작 지점

* JPA는 애플리케이션과 JDBC 사이에서 동작한다. JDBC API 와 DB 사이에 있기 때문에 가운데서 대신 무언가를 해준다.

  ![](https://github.com/namjunemy/TIL/blob/5ab5553422b235a6b06e6dd9e4eedc5b6f14fe21/Jpa/img/01_extends.PNG?raw=true)

* **저장**

  * 위의 상속관계에서 Album 객체를 저장한다고 생각해보자.

  * INSERT 쿼리가 두번 만들어져서 날라간다.

    * INSERT INTO ITEM ...
    * INSERT INTO ALBUM ...

  * JPA persistant 객체에 Album 객체 저장하면. 알아서 INSERT 쿼리 두개 만들어서 넣는다.

  * 단순하게 INSERT 쿼리 두벌 만들어서 DB에 넣는게 아니라 **패러다임의 불일치 자체를 해결**한다.

  * 알아서 두개 클래스에 다 넣어준다.

  * 예)

    ![](https://github.com/namjunemy/TIL/blob/5ab5553422b235a6b06e6dd9e4eedc5b6f14fe21/Jpa/img/02_jpa_save.PNG?raw=true)

* **조회**

  * JPA를 통해서 Album 객체를 조회하게 되면, Item과 Album객체를 이쁘게 조인 쿼리를 날려서 가져오고, Album 객체를 반환한다.

  * 예)

    ![](https://github.com/namjunemy/TIL/blob/5ab5553422b235a6b06e6dd9e4eedc5b6f14fe21/Jpa/img/03_jpa_find.PNG?raw=true)

### JPA 소개

* 예전에 상용 WAS를 사용하던 시절 WAS 1대당 수천만원을 호가했고, 두대정도 깔고 클러스터링 하면 5-6천만원씩 주고 서버를 구성했다. 거기에 EJB라는 기능이 들어있었는데, EJB의 엔티티 빈(자바 표준)이라는 JPA의 초창기 버전이 있었다.
* 이 엔티티 빈의 문제가 너무 많았다. 너무 코드가 지저분하고, API도 너무 복잡하고, 성능도 나빴다. EJB가 너무 문제가 많아서 개빈 킹이 오픈소스 ORM 프레임워크 하이버네이트를 출시했고
* EJB 엔티티 빈의 사용자들이 다 넘어가면서 하이버네이트가 득세를 한다.
* 그 후에 자바진영에서 자바 표준 JPA를 만들었는데, 개빈 킹을 데려와서 만들었다.
* JPA는 사실상 인터페이스만 있다. 구현은 각 기업이 하는거고, 그 중에서 오픈소스인 하이버네이트 같은 구현체들이 존재하는 것이다.
* JPA가 표준화를 거치면서 수 많은 커뮤니티의 용어 등 애매한 것들을 정리하면서 표준을 만들었다.
* JPA는 표준 명세이며
  * 인터페이스의 모음이다
  * JPA 2.1 표준 명세를 구현한 3가지 구현체
    * 하이버네이트, EclipseLink, DataNubleus
* 이 시기에 EJB에 대한 회의감을 느낀 사람이 한명 더 있었는데, Spring 프레임워크를 만든 로드 존슨이다. Spring과 하이버네이트가 득세를 하면서 해외에서는 거의 두가지가 기술 표준이 된다.
* 국내에서는 아쉽게도 Spring을 쓰면서 iBatis/myBatis가 득세를 하게 된다.  5-6년전에 국내에만 90% 사용률을 보였고, 해외해서는 JPA 사용률이 90%였다.

### JPA를 왜 사용해야 하는가?

* SQL 중심적인 개발에서 객체 중심으로 개발

* 생산성

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

* 유지보수

  * 기존에는 필드 변경시 모든 SQL을 수정한다. insert, select, update에 모두
  * 하지만, JPA는 필드만 추가하면 된다. SQL은 JPA가 처리한다.

* 패러다임의 불일치 해결

  * JPA와 상속

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

  * JPA와 연관관계

    * 연관관계 저장

      ```java
      member.setTeam(team);
      jpa.persist(member);
      ```

  * JPA와 객체 그래프 탐색

    ```java
    Member member = jpa.find(Member.class, memberId);
    Team team = member.getTeam();
    ```

    * 뭔가 이상하지 않은가?
    * member만 가져왔는데 어떻게 team이 조회가 되지? NPE 날 것 같은데?
    * 이렇게 예측할 수 있다. JPA가 멤버랑 팀이랑 한방쿼리로 조인해서 가져오나보다.
    * 근데 테이블 50개 연관되어있다. 50개 다 조인해서 가져올까?
    * 그렇지 않다. 아주 우아한 방법으로 동작을 한다.
    * 먼저 답을 말하면 getTeam()은 가능하다. 가져올 수 있다.
    * 방법은 이렇다. member.getTeam()할 때, 정확히 말하면 가져온 team을 사용하는 시점에 team이 가져와져 있지 않으면 그때 가져온다. 팀만 조회하는 쿼리를 날려서. 이것을  **LazyLoding** 이라고 표현한다.
    * 쿼리 두번 나가잖아요! 라고 할 수 있는데, 이게 싫다면 설정을 통해서 한번에 가져오는 쿼리를 날릴 수도 있다.
    * 이렇기 때문에 신뢰할 수 있는 엔티티 계층이 생기게 된다. member.getTeam(), member.getOrder().getDelevery()를 이용해 자유롭게 객체 그래프를 탐색할 수 있다.

  * JPA와 비교하기

    * JPA는 동일 트랜잭션에서 조회한 엔티티는 같음을 보장한다.

      ```java
      String memberId = "100";
      Member member1 = jpa.find(memberId);
      Member member2 = jpa.find(memberId);
      
      member1 == member2; //같다.
      ```

  * JPA를 사용하는 코드를 보면 자바 컬렉션을 사용하는 느낌을 받을 수 있는데, 그게 JPA의 컨셉이다. 나머지는 JPA가 해주겠다는 의미이다.

* JPA의 성능 최적화 기능

  * JPA가 JDBC API와 DB사이에 존재하므로 가운데에 계층이 추가되서 느리지 않을까? 라고 생각할 수 있지만, 반대로 중간에 있으면 할 수 있는게 많다.

    * 1차 캐시와 동일성(identity) 보장

      * JPA와 비교하기의 코드에서 jpa.find()가 두번 일어나는데, JPA는 **같은 트랜잭션에서** 두번째 find는 1차 캐시에서 가져온다.

    * 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)

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

      * 지연로딩 : **객체가 실제 사용될 때** 로딩

        ```java
        Member member = memberDAO.find(memberId);  // SELECT * FROM MEMBER
        Team team = member.getTeam();
        String teamName = team.getName();  //SELECT * FROM TEAM
        ```

      * 즉시로딩: JOIN SQL로 한번에 연관된 객체까지 미리 조회

        ```java
        Member member = memberDAO.find(memberId);  // SELECT ... FROM MEMBER JOIN TEAM ...
        Team team = member.getTeam();
        String teamName = team.getName();
        ```

* ORM은 객체와 RDM 두 기둥위에 있는 기술이다

  * 객체도 잘 다뤄야 하고, RDB도 정말 잘 다뤄야 한다. 실제 현업에서 90% 이상 장애는 DB에서 난다.
  * JPA로 개발하더라도 쿼리가 눈에 다 보여야 한다. 실제로 JPA는 정말 복잡한 쿼리가 아닌 심플한 쿼리들이 나간다.

## Reference

* 자바 ORM 표준 JPA 프로그래밍
* 저자 직강 - <https://www.youtube.com/watch?v=WfrSN9Z7MiA&t=2147s>