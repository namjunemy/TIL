# JPA 연관관계 매핑

'객체지향 설계의 목표는 자율적인 객체들의 협력 공동체를 만드는 것이다' - 조영호(객체지향의 사실과 오해)

## 객체를 테이블에 맞추어 모델링

* 참조 대신에 외래키를 그대로 사용

  ```java
  @Entity
  public class Member {
    @Id
    @GeneratedValue
    private Long id;
    
    @Column(name = "USERNAME")
    private String name;
    
    private int age;
    
    @Column(name = "TEAM_ID")
    private Long teamId;
    ...
  }
  ```

  ```java
  @Entity
  public class Team {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
    ...
  }
  ```

* 외래 키의 식별자를 직접 다뤄야 한다.

  * member.setTeamId(team.getId()); 

    ```java
    public Class Main {
      public static void main(String[] args) {
       
        ...
          
        //팀 저장
        Team team = new Team();
        team.setName("TeamA");
        em.persist(team);
        
        //회원 저장
        Member member = new Memeber();
        member.setName("member1");
        member.setTeamId(team.getId());
        em.persist(member);
      }
    }
    ```

* 저장은 했다고 치자. 
  * 조회를 할 때 두번 따로 조회해야 한다. 연관관계가 없기 때문에 식별자로 다시 팀을 조회한다.

  * 객체 지향적인 방법은 아니다.

    ```java
    //조회
    Member findMember = em.find(Member.class, member.getId());
    
    //연관관계가 없음
    Team findTeam = em.find(Team.class, team.getId());
    ```

* **결론은** 객체를 객체답게 쓰지 못하고 데이터베이스 테이블 스타일에 맞춰서 객체의 관계를 끼워 맞추는 방식을 사용하면
* 즉, 객체를 테이블에 맞추어 데이터 중심으로 모델링하면
* 협력 관계를 만들 수 없다.
  * **테이블은 외래 키로 조인**을 사용해서 연관된 테이블을 찾는다.
  * **객체는 참조**를 사용해서 연관된 객체를 찾는다.
  * 테이블과 객체 사이에는 이런 큰 간격이 존재한다.

## 연관관계 매핑 이론

### 단방향 매핑

* 객체지향 모델링
  * 객체 연관관계 사용

  ![](https://github.com/namjunemy/TIL/blob/master/Jpa/img/06_jpa_relational_mapping.PNG?raw=true)

* 객체의 참조와 테이블의 외래 키를 매핑한다.

  * 외래 키 대신에 TEAM 객체를 넣고
  * TEAM_ID를 매핑한다. 안적어도 default로 들어가지만, 적어 주는게 좋다.
  * 그리고 연관관계를 설정한다 @ManyToOne.
  * 하나의 Team이 여러개의 Member를 가지고 있다. 그렇게 보면 TEAM 입장에서는 일대다!
  * Member 입장에서는 다대일이다. @ManyToOne.
  * 이렇게 설정하면 Team 이라는 변수가 DB에 있는 TEAM_ID라는 FK와 매핑이 된다. 
  * 이것을 **연관관계 매핑**이라고 한다. ORM 매핑!!!!!!!

  ```java
  @Entity
  public class Member {
    @Id
    @GeneratedValue
    private Long id;
    
    @Column(name = "USERNAME")
    private String name;
    
    private int age;
    
  //  @Column(name = "TEAM_ID")
  //  private Long teamId;
    
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    ...
  }
  ```

* 연관관계 저장 방법

  * member.setTeam(team)
  * Team 객체를 그대로 넣어버린다.

  ```java
  //팀 저장
  Team team = new Team();
  team.setName("TeamA");
  em.persist(team);
  
  //회원 저장
  Member member = new Memeber();
  member.setName("member1");
  member.setTeam(team);    //단방향 연관관계 설정, 참조 저장
  em.persist(member);
  ```

* 이렇게 했을때 조회시 Member를 가져와서 getTeam()으로 팀을 바로 조회할 수 있다.

  * 참조로 연관관계 조회 가능. 객체 그래프 탐색!!!!!

    ```java
    //조회
    Member findMember = em.find(Member.class, member.getId());
    
    //참조를 사용해서 연관관계 조회
    Team findTeam = findMember.getTeam();
    ```

* 연관관계 수정

  ```java
  //새로운 팀B
  Team teamB = new Team();
  team.setName("TeamB");
  em.persist(teamB);
  
  //회원1에 새로운 팀B 설정
  member.setTeam(teamB);  
  ```

* 