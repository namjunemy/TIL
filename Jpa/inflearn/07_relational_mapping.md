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

## 양방향 연관관계와 연관관계의 주인

![](https://github.com/namjunemy/TIL/blob/master/Jpa/tacademy/img/07_jpa_relational2.PNG?raw=true)

- Team을 통해서도 getMemberList()로 특정 팀에 속한 멤버 리스트를 가져오고 싶다.
- 객체 설계는 위와 같이 Member에서는 Team을 가지고 있고, Team에서는 Members를 가지고 있도록 설계하면 된다. 
- DB를 보자. DB는 단방향 매핑때와 바뀌는게 없다. 왜냐. 둘을 join 하면 된다. DB는 방향이 없다!
- 이 두가지가 큰 차이다.

### 코드로 이해

* Member 엔티티는 단방향과 동일하다.

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

    * 팀의 입장에서 바라보는일대다, @OneToMany 어노테이션을 설정하고,
    * mappedBy로 team과 연관이 있는 것을 알려준다.
    * 컬렉션을 매핑한다. **관례로 ArrayList로 초기화 한다**. NPE 방지.

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

* 반대 방향으로 객체 그래프 탐색할 수 있다.

    ```java
    //팀 조회
    Team findTeam = em.find(Team.class, team.getId());
    
    // 역방향으로 멤버들 조회
    int memberSize = findTeam.getMembers().size(); 
    ```

### 연관관계 주인과 mappedBy

* 계속 강조하지만, 객체는 방향을 가지고 있고, DB는 방향성이 없다. FK선언 하나면 양방향 관계를 맺을 수 있다.
* mappedBy = JPA의 멘붕 클래스, mappedBy는 처음에는 이해하기 어렵다.
* **객체와 테이블간에 연관관계를 맺는 차이**를 이해해야 한다.

### 객체와 테이블간에 연관관계를 맺는 차이

* **객체 연관관계**

    - 회원 -> 팀 연관관계 1개(단방향)

    - 팀 -> 회원 연관관계 1개(단방향)

    - **객체의 양방향 관계**

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

    - 회원 <-> 팀의 연관관계 1개(양방향)

    - **테이블의 양방향 연관관계**

        - **테이블은 외래 키 하나로 두 테이블의 연관관계를 관리**한다.

        - MEMBER.TEAM_ID 외래 키 하나로 양방향 연관관계를 가진다.(양쪽으로 조인할 수 있다.)

            ```sql
            SELECT *
              FROM MEMBER M
              JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID
             
            SELECT *
              FROM TEAM T
              JOIN MEMBER M ON T.TEAM_ID = M.TEAM_ID
            ```

### 둘 중 하나로 외래 키를 관리해야 한다

- 객체로 보자. 멤버에 팀을 넣고, 팀에 멤버를 넣는다. 두군데 모두 넣어 주는데.
- 둘 중에 뭘로 연관관계를 매핑해야 되나?
    - 멤버의 팀값을 업데이트 할때 FK 업데이트?
    - 팀의 members를 업데이트 했을때 FK 업데이트?
- 하지만 DB 에서는 객체가 참조를 어떻게 하던, FK값만 업데이트 되면 된다.
- 단방향 연관관계 매핑에서는 참조와 외래키만 업데이트 하면 됐지만, 양방향 관계 매핑에서는 다르다.
- 이게 **가장 큰 패러다임의 문제**이다.

![img](https://github.com/namjunemy/TIL/blob/master/Jpa/tacademy/img/08_jpa_fk.PNG?raw=true)

### 연관관계의 주인(Owner)

- 패러다임의 문제를 해결하기 위해 만들어진 개념이다.
- **양방향 매핑 규칙**
    - 객체의 두 관계중 하나를 연관관계의 주인으로 지정
    - 연관관계의 주인만이 외래 키를 관리(등록, 수정) 한다. DB에 접근한다.
        - 객체에 둘다 정보를 업데이트 해도, **연관관계의 주인 것만 DB에 영향**을 준다.
    - **주인이 아닌쪽은 읽기만 가능**
        - **Member에 Team값을 업데이트 하면 mappedBy로 매핑된 Team의 members로 업데이트 된 DB의 멤버들을 읽을 수 있다는 이야기이다.**
        - **Member에 Team 정보 update 했는데, Team에서 getMembers로 가져왔는데 반영 안됐네? -> 연관관계 설정 안했구나.. 라고 바로 알 수 있다.**
        - mappedBy 키워드로 연관관계 매핑이 되었다. 라는 것을 의미한다. 그래서 JoinColumn도 없다.
        - **주인은** mappedBy 속성 사용 하지 않고, @ JoinColumn
        - **주인이 아니면** mappedBy 속성으로 주인을 지정한다.

### 누구를 주인으로 해야 할까?

- **외래 키가 있는 곳을 주인으로 정해라.** @JoinColumn이 있는 쪽.

    - Team.members를 주인으로 선택했다고 가정하고, members를 업데이트 했다. 결과는? Team의 필드값을 바꿨는데, Member로 업데이트 쿼리가 날라간다. ???

- 여기서는 **Member.team**이 연관관계의 주인이다.

- 멤버와 팀이 다대일 관계인데, DB에서 보면 다쪽이 FK를 가지고 있다. 다쪽이 주인이 된다.

- 연관관계의 주인은 비즈니스 적으로 중요하지 않다. 단지 FK를 가진 쪽이 주인이 되면 된다.

- 예를 들면, 자동차와 자동차 바퀴가 있는데 비즈니스적으로는 자동차가 중요하지만, 자동차와 바퀴는 일대다 관계다. 다인 바퀴가 연관관계의 주인이 된다.

- 이렇게 설계 해야,

    - 뒤에서 나올 성능이슈도 없고,
    - 엔티티랑 테이블이 매핑이 되어있는 테이블에서 FK가 관리가 된다.

- 모 민족에서 프로젝트 중에, 모두 단방향으로 설계하고 개발했다. 양방향으로 뚫어서 조회가 필요한 경우에 그 때 양방향 매핑 하면 된다. 자바 코드만 수정이 일어나기 때문에 DB에 영향 주지 않는다.

    - 이미, 단방향 매핑만으로 ORM 매핑이 다 끝났다. 양방향은 단순히 조회를 편하게 하기 위해 부가 기능이 조금더 들어가는 거라고 보면 된다. 어차피 조회할 수 있는 기능만 있다.

    ![img](https://github.com/namjunemy/TIL/blob/master/Jpa/tacademy/img/09_jpa_fk2.PNG?raw=true)

### 양방향 매핑시 가장 많이 하는 실수

* **연관관계의 주인**에 값을 입력하지 않음

* 연관관계의 주인을 Member.team으로 잡아놓고, Team의 members에만 연관관계를 설정한다.

* 주인이 아닌 mappdBy가 설정 되어있는 Team의 members는 조회 권한만 가지고 있고, DB에 영향을 주지 못한다.

* 따라서, **DB에 Member를 조회해보면, TEAM_ID 가 NULL인 상황이 발생한다.**

* 양방향 매핑에 대해서 잘 이해하고 있으면, DB 모델링에 따라서 객체 모델링은, ORM 모델링은 이렇게 엮어 가면 되겠구나 라고 생각하는데 많은 도움이 된다.

    ```java
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);
    
    Member member = new Member();
    member.setName("member1");
    em.persist(member);
    
    //이 작업을 하지 않고,
    //member.setTeam(team);
    
    //역방향(주인이 아닌 방향)만 연관관계 설정
    team.getMembers().add(member);
    
    tx.commit();
    ```

### 양방향 매핑시 주의할점 

* 양방향 매핑시 연관관계의 주인에 값을 입력해야 한다.

* JPA 입장에서만 보면, 연관관계의 주인인 멤버에다가 팀을 세팅해주면 끝난다. DB에 FK값 세팅 되고 문제가 없다.
* 굳이 team의 members 컬렉션에 멤버를 새로 넣어주지 않아도, 지연 로딩을 통해서 해당 멤버를 조회해올 수 있기 때문에 (아래의 코드에서는)아무 문제가 없다.
    * 아래의 코드에서는 em.flush(), clear() 하는 순간에 DB에 FK 세팅 된다. 그래서 지연 로딩을 해도 FK로 조인해서 가져올 수 있다.
    * 하지만, em.flush(), clear()가 일어나지 않으면 DB에 쿼리가 안날라가고, FK도 없이 MEMBER가 1차 캐시에만 영속화 되어있는 상태이다. members 조회해봤자 size 0이다.
* 결론은 진짜 객체 지향적으로 고려하면 항상 **양쪽다 값을 넣어 주는 것이 맞다.**
    * 추가적으로 JPA 없이 순수 자바 Object로 테스트 케이스가 동작하게끔 테스트 코드를 짤때도 NPE 발생한다. 양쪽다 넣어주자.

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");
em.persist(member);

// 연관관계의 주인에 값 설정
member.setTeam(team);

// 역방향 연관관계를 설정하지 않아도, 지연 로딩을 통해서 아래에서 Member에 접근할 수 있다.
//team.getMembers().add(member);

// 이 동작이 수행되지 않으면 FK가 설정되어 있지 않은 1차캐시에만 영속화 된 상태이다. SELECT 쿼리로 조회해봤자 list 사이즈 0이다.
em.flush();
em.clear();

Team findTeam = em.find(Team.class, team.getId());
List<Member> findMembers = findTeam.getMembers();

for (Member m : findMembers) {
    // flush, clear가 일어난 후에는 팀의 Members에 넣어주지 않았지만, 조회를 할 수 있음. 이것이 지연로딩
    System.out.println(m.getUsername());
}

tx.commit();
```

- **순수 객체 상태를 고려해서** 항상 양쪽에 값을 설정하자.

- 연관관계를 설정하다 보면 휴먼에러가 생길 여지가 많다. **연관관계 편의 메소드를 생성하는 것을 권장**한다.

    - 편의 메소드에서는 주인쪽에서 연관관계의 값을 설정할때, 역방향 값도 함께 설정해준다. 메소드를 원자적으로 사용해서 세트로 움직이면 실수할 일이 없어진다.
    - 혹시 편의 메소드 안에 비즈니스 로직이 들어가는 복잡한 경우. 책을 참조해보자.

    ```java
    class Member {
        ...
            
        public void changeTeam(Team team) {
            this.team = team;
            team.getMembers().add(this);
        }
    }
    ```

- **양방향 매핑시에 무한 루프를 주의**하자

    - **lombok이 자동으로 만드는 toString()을 사용하지 말자.**

        - Member의 toString()

            ```java
            @Override
                public String toString() {
                    return "Member{" +
                        "id=" + id +
                        ", username='" + username + '\'' +
                        ", team=" + team +
                        '}';
                }
            ```

        - Team의 toString()

            ```java
            @Override
                public String toString() {
                    return "Team{" +
                        "id=" + id +
                        ", name='" + name + '\'' +
                        ", members=" + members +
                        '}';
                }
            ```

        - Member의 toString()을 호출하는 순간 Team의 toString()의 members가 호출하는 각 member의 toString()때문에 무한루프 생성. 스택오버플로우 발생

    - **JSON 생성 라이브러리**

        - 양방향 관계의 엔티티를 JSON으로 시리얼라이즈 하는순간 무한루프에 빠져버린다.
        - **컨트롤러에서는 엔티티를 절대 직접 반환하지 말자.**
            - 1 - 엔티티 -> JSON 시 무한루프 걸린다.
            - 2 - 엔티티는 충분히 변경의 여지가 있는데, 엔티티 변경시 API 스펙 자체가 변경된다.
                - 실무에서 웬만하면 DTO로 변환해서 반환하자. 그렇게 하면 JSON 생성 라이브러리 때문에 생기는 문제는 없다.

### 양방향 매핑 정리

* **단방향 매핑만으로도 이미 연관관계 매핑은 끝**이다.
    * 실무에서 JPA 모델링 할 때, 단방향 매핑으로 처음에 설계를 끝내야 한다(객체와 테이블을 매핑하는 것). 일대다에서 다쪽에 단방향 매핑으로 쭉 설계하면 이미 테이블 FK 설정은 끝났다. 거기서 필요할때 양방향 매핑을 추가해서 역방향 조회 기능을 쓰면 된다. 테이블 영향없이 자바 코드에 컬렉션만 추가하면 된다.
* 양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가된 것 뿐이다.
* JPQL에서 역방향으로 탐색할 일이 많다. 현업에서 많이 쓴다.
* **단방향 매핑을 잘 하고, 테이블이 굳어진 뒤에 양방향 매핑은 필요할 때 추가해도 된다**. 테이블에 영향주지 않는다.
    * 그동안 학습한 것을 보면, 단방향 매핑에서 양방향 매핑으로 넘어가면서 추가된 것은 팀 객체(엔티티)에 members 컬렉션이 들어간것 뿐이다. 테이블은 똑같다.
* 연관관계의 주인을 정하는 기준은
    * 비즈니스 로직을 기준으로 주인을 선택하면 안된다.
    * **연관관계 주인은 외래 키의 위치를 기준**으로 정해야 한다.

### Reference

- [자바 ORM 표준 JPA 프로그래밍](https://www.inflearn.com/course/ORM-JPA-Basic)

