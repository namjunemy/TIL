# 다양한 연관관계 매핑

### 목차

* 연관관계 매핑시 고려사항 3가지
* 다대일 [N:1]
* 일대다 [1:N]
* 일대일 [1:1]
* 다대다 [N:M]

## 연관관계 매핑시 고려사항 3가지

- 다중성
  - 다대일, 일대다, 일대일, 다대다
- 단방향, 양방향
- 연관관계의 주인

### 다중성

* JPA 에서는 다중성을 위한 어노테이션을 제공한다.
* 이 JPA 어노테이션들은 DB와 매핑하기 위해 존재 한다.
* 그래서 데이터베이스 관점에서의 다중성을 기준으로 고민 하면 된다.
* 다대일 - @ManyToOne
* 일대다 - @OneToMany
* 일대일 - @OneToOne
* 다대다 - @ManyToMany
  * 실무에서 쓰면 안된다.
* 다중성을 고민하다가 풀리지 않으면 대칭성을 고려하자. 다중성의 관계들은 대칭성을 다 가지고 있다. 예를 들면, 회원과 팀 - 팀과 회원 둘다를 고려하면 쉬워 진다.

### 단방향, 양방향

* 이전 학습 내용 정리
* 테이블은
  * 외래 키 하나로 양쪽을 조인할 수 있다.
  * 사실 방향이라는 개념이 없다.
* 객체는
  * 참조용 필드가 있는 쪽으로만 참조할 수 있다.
  * 한쪽만 참조하면 **단방향**이고
  * 양쪽이 서로 참조하면 **양방향**이다.
    * 앞서 말햇듯이 사실은 단방향이 두개가 마주보고 있는 것이다 

### 연관관계의 주인

* 위의 단방향, 양방향의 개념을 알아야. 연관관계의 주인을 이해할 수 있다.
* 테이블은 외래 키 하나로 두 테이블이 연관관계를 맺는데,
* 객체의 양방향 관계는 A->B, B->A 처럼 참조가 2군데에서 일어난다.
* 그렇기 때문에 객체에서 둘중 테이블의 외래키를 관리할 곳을 정해야 한다.
* **연관관계의 주인은 외래 키를 관리하는 참조**이다.
* 주인의 반대편에서는 외래키에 영향을 주지 않고, 단순 읽기만 할 수 있다.

## 다대일 [N:1]

* JPA에서 가장 많이 사용하고, 꼭 알야야 되는 다중성이다.
* 아래 테이블에서 보면, **DB 설계상 일대다에서 다 쪽에 외래키가 있어야 한다**. 그렇지 않으면 잘못된 설계다.
* 객체로 와서, 다대일 관계에서 다인 멤버에서 단순하게 팀으로 단방향 참조를 하고 싶다고 할 때, 테이블에 외래키 있는 쪽인 멤버 객체에서 팀객체로 연관관계 매핑 하면 된다.
* 테이블에서 FK가 팀을 찾기 위해 존재하고, 객체에서 Team 필드도 Team을 참조하기 위해 존재한다.
* 다대일의 반대는 일대다

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/09_modeling.png?raw=true)

* JPA **@ManyToOne으로 다대일 관계 단방향 매핑**

  ```java
  public class Member {
    ...
    
      @ManyToOne
      @JoinColumn(name = "TEAM_ID")
      private Team team;
    
    ...
  }
  ```

### 다대일 양방향 매핑

* 다대일 관계에서 단방향 매핑을 진행하고, 양방향 매핑을 진행할 때
* 반대쪽에서 일대다 단방향 매핑을 해주면 된다.(객체에서 컬렉션 추가해주면 된다.)
* 여기서 중요한건, 반대에서 단방향 매핑을 한다고 해서 DB 테이블에 영향을 전혀 주지 않는다.
* 다대일관계의 **다 쪽에서 이미 연관관계의 주인이 되어서 외래키를 관리**하고 있다.

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/11_many_to_one.PNG?raw=true)

* 반대쪽에서 일대다 단방향 매핑 JPA **@OneToMany** 사용

  * 연관관계의 주인이 아니고, 어디에 매핑 됐는지에 관한 정보 **(mappedBy = "team")**을 꼭 넣어줘야 한다.

  * 그러면 members 필드는 team에 의해 매핑 되어진 것이라는 의미 전달을 확실하게 할 수 있다.

    ```java
    public class Team {
        ...
    
        @OneToMany(mappedBy = "team")
        private List<Member> members = new ArrayList<>();
        
        ...
    }
    ```

* 실제 발생한 쿼리 확인.

  * Main.java
  
    ```java
    Team team = new Team();
    team.setName("A");
    em.persist(team);
    System.out.println("-----팀 저장");
    
    Member member = new Member();
    member.setUsername("NJ");
    member.changeTeam(team);
    em.persist(member);
    System.out.println("-----멤버 저장");
    
    tx.commit();
    ```
    
  * 실제 발생한 쿼리
  
    ```sql
    Hibernate: 
        /* insert hello.jpa.Team
            */ 
            insert 
            into
                Team
                (id, name) 
            values
                (null, ?)
    -----팀 저장
    Hibernate: 
        /* insert hello.jpa.Member
            */ 
            insert 
            into
                Member
                (id, age, createdDate, description, lastModifiedDate, roleType, TEAM_ID, name) 
            values
                (null, ?, ?, ?, ?, ?, ?, ?)
    -----멤버 저장
    ```
  
* 정리

  * 외래키가 있는 쪽이 연관관계의 주인이고,
  * 양쪽을 서로 참조하도록 개발하면 된다.

## 일대다 [1:N]

* 일대다 관계에서는 일이 연관관계의 주인이다.

* 일 쪽에서 외래키를 관리하겠다는 의미가 된다.

* 결론을 먼저 말하자면, 표준스펙에서 지원은 하지만 실무에서 이 모델은 권장하지 않는다.

* **일대다 단방향** 매핑

  ![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/12_one_to_many.PNG?raw=true)

  * 팀과 멤버가 일대다 관계이다.
  * Team이 Members를 가지는데, Member 입장에서는 Team을 참조하지 않아도 된다라는 설계가 나올 수 있다. 객체 입장에서 생각하면 충분히 나올 수 있는 설계이다.
  * 그러나 DB 테이블 입장에서 보면, 무조건 일대다의 다쪽에 외래키가 들어간다.
  * Team에서 members가 바뀌면, DB의 Member 테이블에 업데이트 쿼리가 나가는 상황이다.

* JPA **@OneToMany와 @JoinColumn()을 이용해서 일대다 단방향 매핑** 코드로 이해하기

  * Member는 코드상 연관관계 매핑 없고, 팀에서만 일대다 단방향 매핑 설정.

    ```java
    @Entity
    public class Team {
        ...
        
        @OneToMany
        @JoinColumn(name = "TEAM_ID")
        private List<Member> members = new ArrayList<>();
        
        ...
    }
    ```

  * Main.java

    ```java
    ...
    Member member = new Member();
    member.setUsername("NJ");
    em.persist(member);
    
    System.out.println("-----멤버 저장");
    
    Team team = new Team();
    team.setName("A");
    team.getMembers().add(member);
    em.persist(team);
    
    System.out.println("-----팀 저장");
    
    tx.commit();
    ```

  * 실제 발생한 쿼리

    * log에서 확인해보면 **트랜잭션 커밋 시점**에 create one-to-many row로 시작되는 주석과 함께 Member 테이블을 업데이트 하는 **쿼리가 나간다.**

    ```sql
    Hibernate: 
        /* insert hello.jpa.Member
            */ 
            insert 
            into
                Member
                (id, age, createdDate, description, lastModifiedDate, roleType, name) 
            values
                (null, ?, ?, ?, ?, ?, ?)
    -----멤버 저장
    Hibernate: 
        /* insert hello.jpa.Team
            */
            insert 
            into
                Team
                (id, name) 
            values
                (null, ?)
    -----팀 저장
    Hibernate: 
        /* create one-to-many row hello.jpa.Team.members */ 
        update
            Member 
        set
            TEAM_ID=? 
        where
            id=?
    ```

### 일대다 단방향 정리 - 문제점과 실무적인 해결 방안

* 일대다 단방향은 일대다(1:N)의 일(1)이 연관관계의 주인이다.

* 테이블 일대다 관계는 항상 다(N) 쪽에 외래키가 있다.

* 객체와 테이블의 패러다임 차이 때문에 객체의 반대편 테이블의 외래키를 관리하는 특이한 구조이다.

* @JoinColumn을 꼭 사용해야 한다. 그렇지 않으면 조인 테이블 방식을 사용한다.(중간에 테이블을 하나 추가한다.)

  - 실제로 TEAM_MEMBER라는 테이블이 하나 생성되고, INSERT 쿼리가 나간다.

    ```sql
    Hibernate: 
        
        create table Team_Member (
           Team_id bigint not null,
            members_id bigint not null
        )
        
    ...(생략)
    
    -----팀 저장 이후에
    Hibernate: 
        /* insert collection
            row hello.jpa.Team.members */ 
            insert 
            into
                Team_Member
                (Team_id, members_id) 
            values
                (?, ?)
    ```

* **문제점**

  * 일단 업데이트 쿼리가 나간다. 성능상 좋지않다.
  * Main.java 파일을 보자. 코드를 보면서 쿼리를 추적할 때,
  * 팀 엔티티를 수정했는데 멤버 테이블에 업데이트 쿼리가 나가는 상황이 나오게되고,
  * 실무에서는 테이블이 수십개가 엮여서 돌아가는 상황에서 위와 같은 상황은 운영을 힘들게 한다.

* 답은 위에서 먼저 학습한 다대일 단방향 관계로 매핑하고, 필요할 경우 양방향 매핑을 통해서 해결한다.

* 객체 입장에서보면, 반대방향으로 참조할 필요가 없는데 관계를 하나 만드는 것이지만, DB의 입장으로 설계의 방향을 조금 더 맞춰서 운영상 유지보수하기 쉬운 쪽으로 선택할 수 있다.

### 일대다 양방향 매핑

* 이런 매핑은 공식적으로는 존재하지 않는다.

* @JoinColumn(name = "team_id", insertable = false, updatable = false)

* @ManyToOne과 @JoinColumn을 사용해서 연관관계를 매핑하면, 다대일 단방향 매핑이 되어버린다. 근데 반대쪽 Team에서 이미 일대다 단방향 매핑이 설정되어있다. 이런 상황에서는 두 엔티티에서 모두 테이블의 FK 키를 관리 하게 되는 상황이 벌어진다.

* 그걸 막기 위해서 insert, update를 FALSE로 설정하고 읽기 전용 필드로 사용해서 양방향 매핑처럼 사용하는 방법이다.

* 읽기 전용으로 사용할 경우도 있을 수 있으니, 알아두자.

* **그러나, 결론은 다대일 양방향을 사용하자.**

  * 테이블이 수십개 있는데, 매핑이나 설계는 단순해야 팀원 누구나 사용할 수 있다.

  ```java
  @Entity
  public class Member {
    ...
      
    @ManyToOne
    @JoinColumn(name = "team_id", insertable = false, updatable = false)
    private Team team;
    
    ...
  }
  ```

  