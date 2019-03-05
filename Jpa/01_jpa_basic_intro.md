# JPA Intro

1. SQL  중심적인 개발의 문제점
2. JPA 소개
3. Spring Data PA
4. QueryDSL
5. 실무 경험 공유

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

     * 테이블에 저장할 때에는 `member.getTeam().getId()`로 Id를 조회해서 넣었다.

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

* 데이터 타입

* 데이터 식별 방법