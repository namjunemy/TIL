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