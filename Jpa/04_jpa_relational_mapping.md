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

  ![](https://github.com/namjunemy/TIL/blob/master/Jpa/img/06_jpa_relational.PNG?raw=true)

* 객체의 참조와 테이블의 외래 키를 매핑한다.

  * 외래 키 대신에 TEAM 객체를 넣고
  * TEAM_ID를 매핑한다. 안적어도 default로 들어가지만, 적어 주는게 좋다.
  * 그리고 연관관계를 설정한다 @ManyToOne.
  * 하나의 Team이 여러개의 Member를 가지고 있다.
  * 그렇게 보면 TEAM 입장에서는 일대다,
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

  * 참고 사항

    * JPA 내부에서 1차캐시를 저장하기 때문에 한번 조회하고나면, select 쿼리가 안나가고 캐시에서 리턴한다.
    * 따라서 쿼리를 보려면,
    * em.flush() - DB에 쿼리를 다 날린다.
    * em.clear() - 캐시를 클리어 한다.
    
  * 멤버와 팀의 연관관계를 조회한 쿼리를 보면 애초에 JPA가 조인을 해서 한방쿼리로 가져오는 걸 확인할 수 있다.

  * 하지만, 처음부터 조인해서 가져오지 않고 일단 멤버만 가져오고 싶다!고 하면

  * **@ManyToOne에 패치 옵션을 줄 수 있다.**

    * @**ManyToOne(fetch = FetchType.LAZY)** 
      * 레이지 로딩, 지연로딩. 실제 연관관계를 가진 팀이 Touch 될 때, 그 때 Team을 조회하는 쿼리를 날려서 가져온다.
    * **@ManyToOne(fetch = FetchType.EAGER)**
      * 패치 옵션의 기본값은 EAGER이다. 연관관계를 가진 객체를 조인해서 먼저 다 가져온다.
      * **현업에서는 전부다 LAZY로 기본 설정 해 놓고,** 
      * **꼭 필요한 데서만, 쿼리를 실제 날리는 시점에 원하는 결과를 최적화해서 가져오는 방법을 사용하는 것을 권장한다.**

    ```java
    //캐시 클리어
    em.flush();
    em.clear();
    
    //조회
    Member findMember = em.find(Member.class, member.getId());
    
    //참조를 사용해서 연관관계 조회
    Team findTeam = findMember.getTeam();
    ```

* 연관관계 수정

  * setTeam()으로 새로운 팀을 할당하면, 연관관계가 바뀐다. FK에 들어간 Value가 바뀐다.

  ```java
  //새로운 팀B
  Team teamB = new Team();
  team.setName("TeamB");
  em.persist(teamB);
  
  //회원1에 새로운 팀B 설정
  member.setTeam(teamB);  
  ```

### 양방향 매핑

* Team을 통해서도 getMemberList()로 특정 팀에 속한 멤버 리스트를 가져오고 싶다.
* Member에서는 Team을 가지고 있고, Team에서는 Members를 가지고 있으면 된다. 객체 설계는 아래와 같이.
* DB를 보자. DB는 바뀌는게 없다. 왜냐. 둘을 join 하면 된다. DB는 방향이 없다!
* 이 두가지가 큰 차이다.

![](https://github.com/namjunemy/TIL/blob/master/Jpa/img/07_jpa_relational2.PNG?raw=true)

* Member 엔티티는 단방향과 동일

  ```java
  @Entity
  public class Member {
    @Id
    @GeneratedValue
    private Long id;
    
    @Column(name = "USERNAME")
    private String name;
    
    private int age;
  
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    ...
  }
  ```

* **Team 엔티티는 컬렉션을 추가해주면 된다.**

  * 팀의 입장에서 바라보는 @OneToMany 어노테이션을 설정하고,
  * mappedBy로 team과 연관이 있는 것을 알려준다.
  * 컬렉션을 매핑한다.

  ```java
  @Entity
  public class Team {
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<Member>();
    ...
  }
  ```

* 반대 방향으로 객체 그래프 탐색

  ```java
  //조회
  Team findTeam = em.find(Team.class, team.getId());
  
  int memberSize = findTeam.getMembers().size(); // 역방향 조회
  ```

* 연관관계 주인과 mappedBy

  * 계속 강조하지만, 객체는 방향을 가지고 있고, DB는 방향성이 없다. FK선언 하나면 양방향 관계를 맺을 수 있다.
  * mappedBy = JPA의 멘붕 클래스1
  * mappedBy는 처음에는 이해하기 어렵다.
  * 객체와 테이블간에 연관관계를 맺는 차이를 이해해야 한다.

* **객체와 테이블이 관계를 맺는 차이**

  * **객체 연관관계**

    * 회원 -> 팀 연관관계 1개(단방향)

    * 팀 -> 회원 연관관계 1개(단방향)

    * **객체의 양방향 관계**

      - **객체의 양방향 관계는 사실 양방향 관계가 아니라 서로 다른 단방향 관계 2개**다.

      - 객체를 양방향으로 참조하려면 억지로 단방향 연관관계를 2개 만들어야 한다.

        - A -> B(a.getB())

          ```java
          class A {
            B b;
          }
          ```

        - B -> A(b.getA()) 

          ```java
          class B {
            A a;
          }
          ```

  * **테이블 연관관계**

    * 회원 <-> 팀의 연관관계 1개(양방향)

    * **테이블의 양방향 연관관계**

      - **테이블은 외래 키 하나로 두 테이블의 연관관계를 관리**한다.

      - MEMBER.TEAM_ID 외래 키 하나로 양방향 연관관계를 가진다.(양족으로 조인할 수 있다.)

        ```sql
        SELECT *
          FROM MEMBER M
          JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
         
        SELECT *
          FROM TEAM T
          JOIN MEMBER M ON T.TEAM_ID = M.TEAM_ID
        ```

* **둘 중 하나로 외래 키를 관리해야 한다.**

  * 객체로 보자. 멤버에 팀을 넣고, 팀에 멤버를 넣는다. 두군데 모두 넣어 주는데.
  * 두 객체에 따로따로 두번 넣을 때, DB에는 두번 업데이트 해야할 까?
  * 이게 가장 큰 패러다임 의 문제이다.

![](https://github.com/namjunemy/TIL/blob/master/Jpa/img/08_jpa_fk.PNG?raw=true)

* **연관관계의 주인(Owner)**
  * 이 문제를 해결하기 위해 만들어진 개념이다.
  * **양방향 매핑 규칙**
    * 객체의 두 관계중 하나를 연관관계의 주인으로 지정
    * 연관관계의 주인만이 외래 키를 관리(등록, 수정) 한다. DB에 접근한다.
      * 객체에 둘다 정보를 업데이트 해도, **연관관계의 주인 것만 DB에 영향**을 준다.
    * 주인이 아닌쪽은 읽기만 가능
      * **Member에 Team값을 업데이트 하면 mappedBy로 매핑된 Team의 members로 업데이트 된 DB의 멤버들을 읽을 수 있다는 이야기이다.**
      * **Member에 Team 정보 update 했는데, Team에서 getMembers로 가져왔는데 반영 안됐네? -> 연관관계 설정 안했구나.. 라고 바로 알 수 있다.**
    * mappedBy 키워드로 연관관계 매핑이 되었다. 라는 것을 의미한다. 그래서 JoinColumn도 없지 않은가.
    * **주인은** mappedBy 속성 사용 X !!
    * **주인이 아니면** mappedBy 속성으로 주인을 지정한다.

* 그렇다면, **누구를 주인으로** 해야 할까?

  * **외래 키가 있는 곳을 주인으로 정해라.**

    * Team.members를 주인으로 선택했다고 가정하고, members를 업데이트 했다. 결과는? Member로 쿼리가 날라가는 멘붕이 펼처진다.

  * 여기서는 **Member.team**이 연관관계의 주인이다.

  * 모 민족에서 프로젝트 중에, 모두 단방향으로 설계하고 개발했다. 양방향으로 뚫어서 조회가 필요한 경우에 그 때 양방향 매핑 하면된다. 자바 코드만 수정이 일어나기 때문에 DB에 영향 주지 않는다.

    * 이미, 단방향 매핑만으로 ORM 매핑이 다 끝났다. 양방향은 단순히 조회를 편하게 하기 위해 부가 기능이 조금더 들어가는 거라고 보면 된다. 어차피 조회할 수 있는 기능만 있다.

    ![](https://github.com/namjunemy/TIL/blob/master/Jpa/img/09_jpa_fk2.PNG?raw=true)

* **양방향 매핑시 가장 많이 하는 실수**

  * **연관관계의 주인**에 값을 입력하지 않음

  * 연관관계의 주인을 Member.team으로 잡아놓고, Team의 members에만 연관관계를 설정한다.

  * 이렇게  Team의 members는 되면 조회 권한만 가지고 있고, DB에 영향을 주지 못한다.

  * 따라서, **DB에 Member를 조회해보면, TEAM_ID 가 NULL인 상황이 발생한다.**

  * 양방향 매핑에 대해서 잘 이해하고 있으면, DB 모델링에 따라서 객체 모델링은, ORM 모델링은 이렇게 엮어 가면 되겠구나 라고 생각하는데 많은 도움이 된다.

    ```java
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);
    
    Member member = new Member();
    member.setName("member1");
    
    //이 작업을 하지 않고,
    //member.setTeam(team);
    
    //역방향(주인이 아닌 방향)만 연관관계 설정
    team.getMembers().add(member);
    
    em.persist(member);
    ```

* 양방향 매핑시 연관관계의 주인에 값을 입력해야 한다.

  * 순수한 객체 관계를 고려하면 항상 **양쪽다 값을 입력해야 한다.**

    ```java
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);
    
    Member member = new Member();
    member.setName("member1");
    
    team.getMembers().add(member);
    // 연관관계의 주인에 값 설정
    member.setTeam(team);
    
    em.persist(member);
    ```

* **양방향 매핑의 장점 정리**

  * **단방향 매핑만으로도 이미 연관관계 매핑은 완료**
  * 양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가된 것 뿐
  * JPQL에서 역방향으로 탐색할 일이 많다. 현업에서 많이 쓴다.
  * 단방향 매핑을 잘 하고 양방향은 필요할 때 추가해도 된다. DB 테이블에 영향 주지 않는다.

* 연관관계 매핑 어노테이션

  * 다대일(@ManyToOne)
  * 일대다(@OneToMany)
  * 일대일(@OneToOne)
  * 다대다(@ManyToMany)
    * 현업에서는 제약조건이 많아서 잘 사용하지 않는다. 일대다, 다대일로 풀어내는게 낫다.
  * @JoinColumn, @JoinTable

* 상속 관계 매핑 어노테이션

  * @Inheritance
  * @DiscriminatorColumn
  * @DiscriminatorValue
  * @MappedSuperClass(매핑 속성만 상속)

* 복합키 어노테이션

  * @IdClass
  * @EmbeddedId
  * @Embeddable
  * @MapsId

## Reference

- 자바 ORM 표준 JPA 프로그래밍
- 저자 직강 - <https://www.youtube.com/watch?v=WfrSN9Z7MiA&list=PL9mhQYIlKEhfpMVndI23RwWTL9-VL-B7U>