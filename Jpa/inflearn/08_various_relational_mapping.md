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


## 일대일 [1:1]

* 일대일 관게는 그 반대도 일대일이다.
* 일대일 관계는 특이하게 주 테이블이나 대상 테이블 중에 외래 키를 넣을 테이블을 선택 가능하다.
    * 주 테이블에 외래 키 저장
    * 대상 테이블에 외래 키 저장
* 외래 키에 데이터베이스 유니크 제약조건 추가되어야 일대일 관계가 된다.

### 일대일 - 주 테이블에 외래 키 단방향

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/13_one_to_one.png?raw=true)

* 회원이 딱 하나의 락커를 가지고 있는 상황이다. 반대로 락커도 회원 한명만 할당 받을 수 있는 비즈니스 적인 룰이 있고, 둘의 관계는 일대일 관계이다.
* 이 경우 멤버를 주 테이블로 보고 주 테이블 또는 대상 테이블에 외래 키를 저장할 수 있다. 단, 유니크 제약조건을 추가한 상태에서만.
* **다대일[N:1] 단방향 관계 매핑과 JPA 어노테이션만 달라지고, 거의 유사하다.**

### 일대일 - 주 테이블에 외래 키 양방향

* 다대일[N:1] 양방향 매핑 처럼 **외래키가 있는 곳이 연관관계의 주인**이다.

* JPA @OneToOne 어노테이션으로 일대일 단방향 관계를 매핑하고, @JoinColumn을 넣어준다.

    * @JoinColumn은 Default 값이 있긴 하지만 지저분하게 들어간다. name을 정해주자.

    * 여기까지만 매핑하면 단방향 관계이고

        ```java
        @Entity
        public class Member {
            ...
                
            @OneToOne
            @JoinColumn(name = "locker_id")
            private Locker locker;
         
            ...
        }
        ```

* 반대편에 mappedBy를 적용시켜주면 일대일 양방향 관계 매핑이 된다.

    * mappedBy = "locker" 는 Member엔티티에 있는 Locker 필드와 매핑 되었다는 것을 의미.

    * 이 member 필드는 읽기 전용 필드이다.

        ```java
        @Entity
        public class Locker {
            ...
                
            @OneToOne(mappedBy = "locker")
            private Member member;
        }
        ```

### 일대일 - 대상 테이블에 외래 키 단방향

* 일대일관계에서 대상 테이블에 외래 키를 저장하는 단방향 관계는 JPA에서 지원하지 않는다.

### 일대일 - 대상 테이블에 외래 키 양방향

* 일대일 주 테이블에 외래 키 양방향 매핑을 반대로 뒤집었다고 생각하면 된다. 매핑 방법은 같다.
* 주 테이블은 멤버 테이블이지만, 외래 키를 대상 테이블에서 관리하고 주 테이블의 락커 필드는 읽기 전용이 된다.

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/14_one_to_one.png?raw=true)

### 정리

* 주 테이블에 외래 키
    * (주 테이블 : 많이 접근하는 테이블)
    * 주 객체(많이 사용하는 객체)가 대상 객체의 참조를 가지는 것 처럼
    * 주 테이블에 외래 키를 두고 대상 테이블을 찾는 방식.
    * **객체지향 개발자들이 선호하고, JPA 매핑이 편리하다.**
    * 장점
        * 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인이 가능하다.
    * 단점
        * 값이 없으면 외래 키에 NULL을 허용해야 한다. DB입장에서는 치명적일 수 있다.
* 대상 테이블에 외래키
    * 대상 테이블에 외래 키가 존재한다.
    * 전통적인 데이터베이스 개발자들이 선호하는 방식이다. NULL을 허용해야 하는 문제도 없다.
    * 장점
        * 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조를 유지할 수 있다.(멤버가 락커를 여러개 가지도록 비즈니스 룰이 변경될 경우 테이블 구조 유지하면서 유지보수 가능)
    * 단점
        * 코드상에서는 주로 멤버 엔티티에서 락커를 많이 엑세스 하는데, 어쩔 수 없이 양방향 매핑을 해야한다.
            * 일대일 - 대상 테이블에 외래 키 단방향 매핑을 JPA에서 지원하지 않으므로, 단방향 매핑만 해서는 멤버 객체를 업데이트 했을 때 락커 테이블에 FK를 업데이트 할 방법이 없다. 따라서 양방향 매핑을 해야 한다. 
        * JPA가 제공하는 기본 프록시 기능의 한계로 **지연 로딩으로 설정해도 항상 즉시 로딩** 된다.(프록시는 뒤에서 학습)
            * JPA 입장에서 일대일 관계의 주 테이블에 외래 키를 저장하는 상황에서는, 멤버 객체를 로딩할 때, 멤버 테이블의 FK에 락커 ID가 있는지 없는지만 판단하면 된다. 있으면 프록시 객체를 넣어주고, 없으면 null을 넣으면 된다. 나중에 진짜 락커 필드에 엑세스 할 때, 그때 쿼리가 나간다.
            * 그런데, 대상 테이블에 외래 키를 저장한다면, JPA가 멤버의 락커를 조회하는 상황에서 DB의 멤버 테이블만 조회해서는 모른다. 어차피 락커 테이블을 찾아서 멤버가 있는지 확인 해야(쿼리를 날려 봐야) 알 수 있다. 어차피 쿼리가 나간단 이야기는 프록시를 만들 필요가 없다는 이야기이다. 그래서 하이버네이트 구현체 같은 경우는 지연 로딩으로 설정해도 항상 즉시 로딩 된다.