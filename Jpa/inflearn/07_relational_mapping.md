# 연관관계 매핑 기초

### 학습 목표

* 객체와 테이블 연관관계의 차이를 이해한다.
* 객체의 참조와 테이블의 외래 키를 어떻게 매핑하는지 안다.
* 용어 이해
    * 방향(Direction)
        * 단방향, 양방향
    * 다중성(Multiplicity)
        * 다대일, 일대다, 일대일, 다대다 이해
    * 연관관계의 주인(Owner)
        * 객체 양방향 연관관계는 관리하는 주인이 필요함

### 연관관계가 필요한 이유

* '객체지향 설계의 목표는 자율적인 객체들의 협력 공동체를 만드는 것이다' - 조영호(객체지향의 사실과 오해)

### 예제 시나리오

* 회원과 팀이 있다.
* 회원은 하나의 팀에만 소속될 수 있다.
* 회원과 팀은 다대일 관계다.

## 객체를 테이블에 맞추어 모델링하면..

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/08_relation.png?raw=true)

- 연관관계가 없는 객체, **참조 대신 외래 키를 그대로 사용한다.**

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

- 외래 키의 식별자를 직접 다뤄야 한다.

    - 팀을 저장하고, 회원을 저장하면서 팀 ID(외래키 식별자)를 직접 회원 객체에 set 한다.

    - **member.setTeamId(team.getId());** 

        ```java
        public Class Main {
          public static void main(String[] args) {
           
            ...
              
            //팀 저장
            Team team = new Team();
            team.setName("TeamA");
            em.persist(team);	// 영속 상태가 됨. PK값이 세팅 된 상태.
            
            //회원 저장
            Member member = new Memeber();
            member.setName("member1");
            member.setTeamId(team.getId());	// PK값이 세팅 됐으므로, getId() 가능.
            em.persist(member);
          }
        }
        ```

- 저장은 했다고 치자. 

    - 조회를 할 때 두 번 따로 조회해야 한다. 연관관계가 없기 때문에 식별자로 다시 팀을 조회한다.

    - 객체 지향적인 방법은 아니다.

        ```java
        //조회
        Member findMember = em.find(Member.class, member.getId());
        
        //연관관계가 없음
        Team findTeam = em.find(Team.class, team.getId());
        ```

- 객체를 객체답게 쓰지 못하고 데이터베이스 테이블 스타일에 맞춰서 객체의 관계를 끼워 맞추는 방식을 사용하면,

- 즉, 객체를 테이블에 맞추어 데이터 중심으로 모델링하면 **협력 관계를 만들 수 없다.**

    - **테이블은 외래 키로 조인**을 사용해서 연관된 테이블을 찾는다.
    - **객체는 참조**를 사용해서 연관된 객체를 찾는다.
    - 테이블과 객체 사이에는 이런 큰 간격이 존재한다.

## 단방향 연관관계 매핑 이론

### 객체지향 모델링

- 객체 연관관계 사용해서 아래의 객체 참조와 테이블의 외래키를 매핑하는 방법 학습

![img](https://github.com/namjunemy/TIL/blob/master/Jpa/tacademy/img/06_jpa_relational.PNG?raw=true)

- **객체의 참조와 테이블의 외래 키를 매핑**한다.

    - 외래 키 대신에 TEAM 객체를 넣고
    - TEAM_ID를 매핑한다. @JoinColumn으로 조인 컬럼을 명시한다. 안적어도 default로 들어가지만, 적어 주는게 좋다.
    - 그리고 연관관계를 설정한다 @ManyToOne.
    - 하나의 Team이 여러개의 Member를 가지고 있다.
    - 그렇게 보면 TEAM 입장에서는 일대다,
    - Member 입장에서는 다대일이다. @ManyToOne.
    - 이렇게 설정하면 Team 이라는 필드가 DB에 있는 TEAM_ID라는 FK와 매핑이 된다. 
    - **관계를 선언하고 조인할 컬럼을 매핑 했다.**
    - 이것을 **연관관계 매핑**이라고 한다. ORM 매핑!!!!!!!

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

### 객체의 참조와 테이블의 외래키를 매핑한 후의 모델링(ORM 매핑)

 ![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/09_modeling.png?raw=true)

### 연관관계 저장 방법

- member.setTeam(team)
- Team 객체를 그대로 넣어버린다.

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

Member findMember = em.find(Member.class,  member.getId());
```

- 이렇게 했을때 조회시 Member를 가져와서 getTeam()으로 팀을 바로 조회할 수 있다.

    - 참조로 연관관계 조회 가능. 객체 그래프 탐색!!

    - 물론, 위의 em.find()시 select 쿼리는 날라가지 않는다.

        - JPA 내부에서 1차캐시를 저장하기 때문에 한번 조회하고나면, select 쿼리가 안나가고 캐시에서 리턴한다.

    - 멤버와 팀의 연관관계를 조회한 쿼리를 보면 애초에 JPA가 조인을 해서 한방쿼리로 가져오는 걸 확인할 수 있다.

    - 하지만, 처음부터 조인해서 가져오지 않고 일단 멤버만 가져오고 싶다라고 하면

    - **@ManyToOne에 패치 옵션을 줄 수 있다.**

        - @**ManyToOne(fetch = FetchType.LAZY)** 

            - 레이지 로딩, 지연로딩. 실제 연관관계를 가진 팀이 Touch 될 때, 그 때 Team을 조회하는 쿼리를 날려서 가져온다.

        - **@ManyToOne(fetch = FetchType.EAGER)**

            - 패치 옵션의 기본값은 EAGER이다. 연관관계를 가진 객체를 조인해서 먼저 다 가져온다.

            - **현업에서는 전부다 LAZY로 기본 설정 해 놓고,** 

            - **꼭 필요한 데서만, 쿼리를 실제 날리는 시점에 원하는 결과를 최적화해서 가져오는 방법을 사용하는 것을 권장한다.**

                ```java
                //캐시 클리어해서 find시 날라가는 쿼리 확인 가능
                em.flush();
                em.clear();
                
                //조회
                Member findMember = em.find(Member.class, member.getId());
                
                //참조를 사용해서 연관관계 조회
                Team findTeam = findMember.getTeam();
                ```

### 연관관계 수정

- setTeam()으로 새로운 팀을 할당하면, 연관관계가 바뀐다.
- Update 쿼리를 통해서 FK에 들어간 Value가 바뀐다.

```java
//새로운 팀B
Team teamB = new Team();
team.setName("TeamB");
em.persist(teamB);

//회원1에 새로운 팀B 설정
member.setTeam(teamB);
```

