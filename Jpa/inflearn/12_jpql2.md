# JPA 객체지향 쿼리 언어 - 중급문법

## 경로 표현식

* .(점)을 찍어 객체 그래프를 탐색하는 것

    ```java
    select m.username -> 상태 필드
      from Member m
      join m.team t   -> 단일 값 연관 필드
      join m.orders o -> 컬렉션 값 연관 필드
     where t.name = '팀A'
    ```

### 경로 표현식 용어 정리

* 상태 필드(state field)
    * **단순히 값을 저장** 하기 위한 필드(ex: m.username)
* 연관 필드(association field)
    * 연관관계를 위한 필드
    * **단일 값 연관 필드**
        * @ManyToOne, @OneToOne, **대상이 엔티티** (ex: m.team)
    * **컬렉션 값 연관 필드**
        * @OneToMany, @ManyToMany, **대상이 컬렉션** (ex: m.orders)

### 경로 표현식 특징

* **상태 필드(state field)**

    * 경로 탐색의 끝. 더 이상 탐색할 수 없다.
    * 더 이상 탐색할 수 없다는 이야기는
    * select m.username 뒤에 . 을 찍어서 탐색 경로를 표현할 수 없다는 이야기이다.

* **단일 값 연관 경로**

    * **묵시적 내부 조인(inner join) 발생**. 더 탐색이 가능하다.

        * 객체 입장에서야 .을 찍어서 필드에 접근하면 되지만, DB 입장에서는 조인이 필요하다. 묵시적 조인이 발생한다.
        * 묵시적인 내부 조인이 발생하도록 웬만하면 코드 짜지 말자.
        * 실무에서는 조인 쿼리가 수십개 나가는데, 조인은 성능 튜닝에 막대한 영항을 준다. 어디서 조인 쿼리 나갔는지 찾기도 어렵다.
        * 따라서 어떤식으로 조인하면서 쿼리를 질의하냐가 중요한데, 묵시적으로 조인 쿼리가 나가버리면 사이즈가 큰 어플리케이션 바탕으로 실무하면서 좋지 않다.
        * 최대한 JPQL과 SQL을 맞춰서 직관적으로 코딩하자.

        ```java
        String query =
        "select m.team from Member m";
        
        List<Team> result = em.createQuery(query, Team.class)
        .getResultList();
        ```

        * 묵시적으로 발생한 내부 조인 쿼리

            ```sql
            Hibernate: 
                /* select
                    m.team 
                from
                    Member m */ 
                  	select
                        team1_.id as id1_3_,
                        team1_.name as name2_3_ 
                    from
                        Member member0_ 
                    inner join
                        Team team1_ 
                            on member0_.team_id=team1_.id
            ```

    * select m.team.name 이 가능하다는 이야기이다. team 뒤에 . 을 찍어서 추가 탐색이 가능하다.

    * team.name은 상태 필드 이므로 경로 탐색의 끝이다.

* **컬렉션 값 연관 경로**

    * 묵시적 내부 조인 발생. 하지만, 더 이상 탐색할 수 없다.

        * t.members로 경로 표현하는 순간 컬렉션이다. 컬렉션 자체이므로 필드에 접근하거나 더 이상 탐색할 수 없다. 대신에 컬렉션의 메서드인 size()는 사용 가능하다.

            ```java
            String query = "select t.members.size from Team t";
            ```

            ```java
            String query =
                "select t.members from Team t";
            
            List result = em.createQuery(query, Collection.class)
                .getResultList();
            ```

        * 묵시적으로 발생한 내부 조인 쿼리

            ```sql
            Hibernate: 
                /* select
                    t.members 
                from
                    Team t */ 
                    select
                        members1_.id as id1_0_,
                        members1_.age as age2_0_,
                        members1_.memberType as memberTy3_0_,
                        members1_.name as name4_0_,
                        members1_.team_id as team_id5_0_ 
                    from
                        Team team0_ 
                    inner join
                        Member members1_ 
                            on team0_.id=members1_.team_id
            ```

    * FROM 절에서 **명시적 조인을 통해 별칭을 얻으면 별칭을 통해 탐색 가능**하다.

        ```java
        String query =
            "select m.name from Team t join t.members m";
        
        List<String> result = em.createQuery(query, String.class)
            .getResultList();
        
        System.out.println(result); //[관리자1, 관리자2]
        ```

        ```sql
        Hibernate: 
            /* select
                m.name 
            from
                Team t 
            join
                t.members m */ 
                select
                    members1_.name as col_0_0_ 
                from
                    Team team0_ 
                inner join
                    Member members1_ 
                        on team0_.id=members1_.team_id
                        
        [관리자1, 관리자2]
        ```

* 정리

    * 묵시적 조인은 실무에서 절대 사용하지 말자.
    * 명시적 조인으로 명확하게 관리하자.

### 명시적 조인, 묵시적 조인

* 명시적 조인 

    * join 키워드 직접 사용

        ```sql
        select m from Member m join m.team t
        ```

* 묵적 조인

    * 경로 표현식에 의해 묵시적으로 SQL 조인 발생(내부 조인만 가능)

        ```sql
        select m.team from Member m
        ```

### 경로 표현식 - 예제

* SELECT o.member.team FROM Order o
    * 성공
    * 단, 문제는 묵시적인 내부 조인이 2회 발생.
    * 경로 표현식은 쉽고 단순한 것 처럼 보이나, 어떤 SQL이 실행될지는 예측하기 어려움
* SELECT t.members FROM Team
    * 성공
* SELECT t.members.username FROM Team t
    * 실패
* SELECT m.username FROM Test t Join t.members m
    * 성공

### 경로 탐색을 사용한 묵시적 조인시 주의 사항

* 항상 내부 조인이 발생
* 컬렉션은 경로 탐색의 끝, 명시적 조인을 통해 별칭을 얻어야 한다.
* 경로 탐색은 주로 SELECT, WHERE 절에서 사용하지만 묵시적 조인으로 인해 **SQL의 FROM (JOIN) 절에 영향을 줌**

### 실무 조언

* 가급적 묵시적 조인 대신에 명시적 조인을 사용한다.
* 조인은 SQL 튜닝에 중요한 포인트 이므로, 기승전 DB. 명시적으로 가져가자.
* 묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어렵다.

## JPQL fetch join 1 - 기본

* SQL 조인의 종류가 아니다.
* JPQL에서 **성능 최적화** 를 위해 제공하는 기능이다.
* 연관된 엔티티나 컬렉션을 **SQL 한번에 함께 조회** 하는 기능이다.

* join fetch 명령어를 사용한다.
* [ LEFT [OUTER] | INNER ] JOIN FETCH 조인경로

### 엔티티 페치 조인

* 회원을 조회하면서 연관된 팀도 함께 조회하고 싶다.(SQL 한번에)

* SQL을 보면 회원 뿐만 아니라 팀(T.*)도 함께 SELECT 한다.

* JPQL

    ```sql
    select m from Member m join fetch m.team
    ```

* 실제 실행된 SQL - **join 발생**.

    ```sql
    SELECT M.*, T.* 
    FROM MEMBER M 
    INNER JOIN TEAM T ON M.TEAM_ID = T.ID
    ```

* 예제

    * 아래에서 @ManyToOne 지연 로딩 설정된 Member의 Team에 접근시 **1+N 쿼리가 발생** 한다.
    * Member를 3개 가져오고, 루프를 돌면서 getTeam().getName()으로 접근하는 순간 TEAM 정보를 조회하는 쿼리가 하나씩 나간다.
    * fetch join 사용시 지연 로딩을 하지 않고, 회원과 팀을 한번에 조회한다.

    ```java
    String query = "select m from Member m";
    String fetchJoinQuery = "select m from Member m join fetch m.team";
    
    List<Member> result = em.createQuery(fetchJoinQuery, Member.class)
        .getResultList();
    
    result.forEach(m -> System.out.println(m.getName() + ":" + m.getTeam().getName()));
    
    ```

### 컬렉션 페치 조인

* 일대다 관계, 컬렉션 페치 조인

* JPQL

    ```sql
    select t 
    from Team t join fetch t.members
    where t.name = '팀A'
    ```

* 실제 실행된 SQL - **join 발생**.

    ```sql
    SELECT T.*, M.* 
    FROM TEAM T 
    INNER JOIN MEMBER M ON T.ID = M.TEAM_ID
    WHERE T.NAME = '팀A'
    ```

* 예제

    ```java
    String query = "select t from Team t join fetch t.members";
    
    List<Team> result = em.createQuery(query, Team.class)
        .getResultList();
    
    for (Team t : result) {
        System.out.println(t.getName() + ":" + t.getMembers().size());
        for (Member m : t.getMembers()) {
            System.out.println("--> " + t.getName() + ":" + m.getName());
        }
    }
    ```

    * 결과

        * 원하는대로 데이터가 찍히는데, 이상하다. 
        * 팀은 2개인데 결과는 3개가 찍힌다. 뭔가 더 찍힌다. 주의할 점에 대해서 알아보자.

        ```sql
        Hibernate: 
            /* select
                t 
            from
                Team t 
            join
                fetch t.members */ 
                select
                    team0_.id as id1_3_0_,
                    members1_.id as id1_0_1_,
                    team0_.name as name2_3_0_,
                    members1_.age as age2_0_1_,
                    members1_.memberType as memberTy3_0_1_,
                    members1_.name as name4_0_1_,
                    members1_.team_id as team_id5_0_1_,
                    members1_.team_id as team_id5_0_0__,
                    members1_.id as id1_0_0__ 
                from
                    Team team0_ 
                inner join
                    Member members1_ 
                        on team0_.id=members1_.team_id
        팀A:2
        --> 팀A:회원1
        --> 팀A:회원2
        팀A:2
        --> 팀A:회원1
        --> 팀A:회원2
        팀B:1
        --> 팀B:회원3
        ```

* **일대다 관계에서 페치 조인시 주의할 점**

    * 아래 DB테이블 관점에서 보면, 
    * 팀 A 입장에서는 MEMBER row 2개와 조인을 하게 된다.
    * 가운데 있는 [TEAM JOIN MEMBER] 테이블과 같은 결과가 만들어진다.
    * 위의 실습에서 데이터가 많이 찍힌 이유가 여기에 있다.
    * 팀 A에 회원이 100명 있었다면, 팀A에 대한 결과가 100개 찍혔을 것이다.
    * 일대다 조인 쿼리를 통해서 가져오는 동작에서는 JPA는 무언가를 할 수 없다. DB에서 조회하면 그대로 가져와야 한다.
    * 맨 아래의 그림에서 보면, teams결과 리스트는 2개 이지만 팀A는 영속성 컨텍스트가 보장하는 동일한 객체이다.

    ![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/40_fetch_join.png?raw=true)

### 페치 조인과  DISTINCT

* SQL의 DISTINCT는 중복된 결과를 제거하는 명령이다.

* JPQL의 DISTINCT는 2가지 기능을 제공한다.

    * 1 - SQL에 DISTINCT를 추가하거나
    * 2 - **애플리케이션에서!** 엔티티 중복을 제거해준다.

* 예제

    * 위의 컬렉션 페치조인 예제에서  distinct 명령어를 사용해서 조회해보자.

    * 결과를 보면 줄었다. 원하는대로 팀 2개에 결과 2개가 찍힌다.

        ```java
        String query = "select distinct t from Team t join fetch t.members";
        
        List<Team> result = em.createQuery(query, Team.class)
            .getResultList();
        
        for (Team t : result) {
            System.out.println(t.getName() + ":" + t.getMembers().size());
            for (Member m : t.getMembers()) {
                System.out.println("--> " + t.getName() + ":" + m.getName());
            }
        }
        ```

        ```java
        Hibernate: 
            /* select
                distinct t 
            from
                Team t 
            join
                fetch t.members */ select
                    distinct team0_.id as id1_3_0_,
                    members1_.id as id1_0_1_,
                    team0_.name as name2_3_0_,
                    members1_.age as age2_0_1_,
                    members1_.memberType as memberTy3_0_1_,
                    members1_.name as name4_0_1_,
                    members1_.team_id as team_id5_0_1_,
                    members1_.team_id as team_id5_0_0__,
                    members1_.id as id1_0_0__ 
                from
                    Team team0_ 
                inner join
                    Member members1_ 
                        on team0_.id=members1_.team_id
        
        팀A:2
        --> 팀A:회원2
        --> 팀A:회원1
        팀B:1
        --> 팀B:회원3
        ```

* SQL 쿼리에서보면 distinct가 추가 됐다. 하지만 SQL은 완전히 동일한 row만 중복을 제거한다.

* JPQL의 DISTINCT가 추가로 **애플리케이션에서 중복 제거를 시도한다.**

* 같은 식별자를 가진 **Team 엔티티를 제거한다.** 따라서, 원했던 결과를 얻을 수 있다.

* 일대다 조인은 다의 수 만큼 데이터가 뻥튀기 된다.

    ![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/41_fetch_join.png?raw=true)

### 페치 조인과 일반 조인의 차이

* 일반 조인 실행시 연관된 엔티티를 함께 조회하지 않는다.

    * 실행된 SQL에서 SELECT문을 보면 team의 정보들만 조회한다.
    * JPQL은 결과를 반환할 때 연관관계를 고려하지 않는다.
    * 단지 SELECT 절에 지정한 엔티티만 조회할 뿐이다
    * team의 members는 초기화 되지 않은 상태라서, 접근시 쿼리가 추가로 나간다.

    ```java
    String query = "select t from Team t join t.members";
    
    List<Team> result = em.createQuery(query, Team.class)
                    .getResultList();
    ```

    ```sql
    Hibernate: 
        /* select
            t 
        from
            Team t 
        join
            t.members */ select
                team0_.id as id1_3_,
                team0_.name as name2_3_ 
            from
                Team team0_ 
            inner join
                Member members1_ 
                    on team0_.id=members1_.team_id
    ```

* 페치 조인을 사용할 때만 연관된 엔티티도 함께 조회 한다.(즉시 로딩)

* **페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념이다. 실무에서 정말 많이 쓰인다.**