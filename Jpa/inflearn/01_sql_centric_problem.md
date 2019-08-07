# 1. SQL 중심적인 개발의 문제점

현재 데이터베이스 세계의 헤게모니를 관계형 DB가 가지고 있다(Oracle, MySQL, ...). 지금 시대에는 객체를 관계형 DB에 관리하고 있다는 이야기이다. 관계형 DB를 사용하려면 SQL을 짜야한다. 계속, SQL 중심적인 개발을 하게되면 아래와 같은 문제점이 있다.

## 무한 반복, 지루한 코드

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

  * 추가 하는 과정에서 누락이 있다면, 헬게이트 오픈이다.
  
  * **결국, 우리가 관계형 DB를 쓰는 상황에서는 SQL에 의존적인 개발을 피하기 어렵다**
  
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

## 엔티티 신뢰 문제

* 코드를 객체 지향적으로 짰다고 가정하고, MemberDAO에서 member를 하나 꺼내왔다고 하자.
* 해당 멤버의 팀과 주문정보, 그 주문정보의 배송지 정보를 getter로 가져오려고 하는 건 자연스러운 생각이다.
* 하지만, 해당 member가 팀과 주문과 배송지 정보를 가지고 있다고 보장되지 않는 이상 이렇게 코드를 짤 수 없다.
* DAO의 코드를 까서 팀과 주문정보 주문에 대한 배송지 정보들을 쿼리로 정말 가져오는지 눈으로 확인해봐야 한다.
* 계층형 아키텍처(controller, domain, service등의 계층형 구조)의 진정한 의미의 계층 분할이 어렵다는 이야기이다. 즉, 물리적으로는 분리되어 있지만 논리적으로는 전혀 분할되어 있지 않다는 이야기 이다.
* **결과적으로 어떤 쿼리를 짜느냐에 따라 비즈니스 로직에 영향을 주기 때문에 SQL 의존적인 개발을 피하기 어렵다.**

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

## 패러다임의 불일치

* 객체 vs 관계형 데이터베이스
  * **관계형 데이터베이스**는 철저히 '데이터를 어떤식으로 잘 저장할까'에 포커스가 맞춰져있다.
  * **객체**는 데이터 저장이 아니라 '어떻게 더 추상화하고 관리를 잘 할까'가 더 중요하다.
  * 우리는 포커스가 다른 두 가지를 억지로 맵핑해서 일을 처리해야하기 때문에 그 사이에서 많은 시간을 보내고 있다.
* 객체를 영구 보관하는 다양한 저장소
  * Object -> RDB, NoSQL, File 등 객체를 저장할 수 있는 방법이 많이 있지만,
  * **현실적인 대안은 관계형 데이터베이스이다.**
* 객체를 관계형 DB에 저장하려면
  * 객체를 SQL로 바꾼다. (개발자가, 한땀한땀)
  * 개발자가 SQL 매퍼의 일을 너무 많이 하고 있다.

## 객체와 관계형 데이터베이스의 차이

### 상속

* 객체의 상속 관계
* Table 슈퍼타입 서브타입 관계
    * 객체 상속관계와 그나마 유사한 논리 모델
    * 아래는 물리 모델로 잘 표현해놓은 관계도이다.

![](https://github.com/namjunemy/TIL/blob/master/Jpa/tacademy/img/01_extends.PNG?raw=true)

* Album객체를 **저장**하는 경우에
    * 객체를 분해 한다. item을 상속받았으니까 앨범이 데이터를 다 가지고 있을 것이다.
    * 그리고 DB에 저장하려고하면 다른 INSERT 쿼리를 두번 날려야 한다.
        * INSERT INTO ITEM ...
        * INSERT INTO ALBUM ...
    * 저장은 그래도 어떻게든 했다고 치자
* Album을 **조회**하려고 하면?
    * 각각의 테이블에 따른 조인 SQL을 작성한다.
        * ALBUM과 ITEM, MOVIE와 ITEM 조인해서 데이터 가져온다.
    * 각각의 객체 생성해서 필드에 넣어주고.....상상만해도 복잡하다.
    * **그래서 DB에 저장할 객체에는 상속 관계를 안쓴다.**
* 근데, DB가 아니고 **자바 컬렉션에 저장**한다고 생각해보자
    * 그냥 `list.add(album);` 으로 컬렉션에 넣는다.
* 자바 컬렉션에서 조회하면?
    * `Album album = list.get(albumId);` id로 꺼내오면 된다.
    * 심지어 객체 세상이기 때문에 필요하면 부모 타입으로 조회 후 다형성 활용도 가능하다.
        * `Item item = list.get(albumId);`
* **자바 컬렉션에서는 굉장히 심플한 작업이, 관계형 DB에 넣고 빼는 순간 중간의 매핑 작업을 개발자가 해줘야 되므로 매우 번잡한 일이 된다.**

### 연관관계

![](https://github.com/namjunemy/TIL/blob/master/Jpa/tacademy/img/06_jpa_relational.PNG?raw=true)

* **객체**는 **참조**를 사용

  * member.getTeam()
  * 객체의 연관관계에는 **방향성이 있다**. 멤버에서 팀을 조회할 수 있지만, 팀에서 멤버조회는 불가하다.
    * Member의 필드는 id, Team, username
    * Team의 필드는 id, name 이라고 가정.

* **테이블**은 **외래 키**를 사용

  * FK와 PK를 조인해서 조회
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

* 이 때, 위의 **객체를 테이블 설계에 맞추어 모델링**하게 되면 아래와 같이 FK를 그대로 필드로 포함하게 된다. 하지만, Member클래스에 Team 객체가 존재하는 것이 더 객체지향적이다 라고 할 수 있다. FK의 값을 넣는것 보단.

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

   * **객체를 테이블에 저장**할 때에는 `member.getTeam().getId()`로 teamId를 조회해서 넣었다.

      ```sql
      INSERT INTO MEMBER(MEMBER_ID, TEAM_ID, USERNAME) VALUES ...
      ```

   * 그러나, **조회를 하려고 하면…. 헬게이트 오픈**이다. 먼저 멤버와 팀을 조인해 놓고 팀을 조회할 준비를 한다.

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

* **(객체지향 적인)객체 모델링을 자바 컬렉션에 관리**한다고 생각하면, 객체지향 적인 설계가 괜찮은 설계가 된다.
* 리스트에 멤버를 저장하면, 팀도 같이 저장된다.
  
    * `list.add(member);`
* 멤버가 필요하면?
  
    * `Member member = list.get(memberId);`
* 멤버의 팀을 조회하고 싶으면?
  
    * `Team team = member.getTeam();`
* 이걸 RDB에 저장하고 조회하면 생산성이 안나오기 시작한다.
  
  * 그래서 슈퍼 DTO 만들어서 객체 하나로 반환해서 사용했다. 그게 생산성 측면에서 더 좋았다.

### 객체 그래프 탐색

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/01_graph_navigation.png?raw=true)

* 객체는 자유롭게 객체 그래프를 탐색할 수 있어야 한다.

  * 예를 들면, member.getTeam(), member.getOrder(), member.getOrder().getOderItem() 할 수 있어야 된다.

* DB에서는 멤버와 팀, 멤버와 오더 서로 조회를 할 수 있다. 하지만, 서비스 로직에서는 못한다. 왜? 처음에 쿼리에 넣어놓지 않아서 가져오지 않았기 때문에.

* 핵심은 **처음 실행하는 SQL에 따라 탐색 범위가 결정**된다는 것이다.

  * 쿼리에서 오더를 가져오지 않았기 때문에 getOrder()는 NULL이다.

  ```sql
  SELECT M.*, T.*
    FROM MEMBER M
    JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
  ```

  ```java
  member.getTeam(); //OK
  
  member.getOrder(); //NULL
  ```

* **결국 엔티티 신뢰 문제**가 발생하게 된다.

    * 아래와 같은 서비스 로직을 짤 때, 내가 DAO를 작성하지 않았다고 생각하면 **DAO에서 반환된 엔티티를 신뢰하고 사용할 수 없다**. 
    * memberDAO가 member를 어떻게 가져오느냐 눈으로 확인해야된다. NPE 여지가 다분하다.
    * 일반적으로 Layered 아키텍처에서는 그 다음 계층에서 신뢰를 하고 사용해야 하는데, 여기에서는 엔티티 계층을 신뢰할 수 없는 문제가 발생하게 된다.
    * 물리적으로는 서비스, DAO 등의 층이 분리되어 있지만, 논리적으로는 DAO 내부를 열어 보고 직접 가져오는지 확인하는 등의 작업이 필요하므로 다 엮여 있는 상태이다.

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
      * getMember();
      * getMemberWithTeam();
      * getMemberWithOrderWithDelivery();
  * 이게 맞을까?
  * **결론적으로 SQL을 직접 다루게 되면, 진정한 의미의 계층 분할이 어렵다.**

### 비교하기

* **일반적인 SQL을 사용하는 경우**

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

* **자바 컬렉션에서 조회하는 경우**

    * 그런데, 자바 컬렉션에서 조회한다고 가정해보면 두 멤버는 같은 멤버이다.

        ```java
        String memberId = "100";
        Member member1 = list.get(memberId);
        Member member2 = list.get(memberId);
        
        member1 == member2; //같다.
        ```

## 마무리

* 위의 비교들을 통해 둘간의 패러다임 차이를 느낄 수 있다.
* 결론적으로 객체답게 모델링 할수록 매핑 작업만 늘어나게 된다.
* 객체를 자바 컬렉션에 저장하듯이 DB에 저장할 수는 없을까?
* 자바진영에서는 그 고민의 결과가 **JPA**다.

### Reference

* [자바 ORM 표준 JPA 프로그래밍](https://www.inflearn.com/course/ORM-JPA-Basic)