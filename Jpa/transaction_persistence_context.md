## JPA의 트랜잭션 범위의 영속성 컨텍스트 전략과 스프링이 엔티티 매니저의 Thread-safe를 보장하는 방법

* 스프링이나 J2EE 컨테이너 환경에서 JPA를 사용하면 컨테이너가 트랜잭션과 영속성 컨텍스트를 관리해주므로 애플리케이션을 손쉽게 개발할 수 있다. 하지만, 내부를 모르면 트러블슈팅을 할 수가 없으므로 잘 알아두자.

* 스프링 컨테이너는 트랜잭션 범위의 영속성 컨텍스트 전략을 기본으로 사용한다.

  * 이 전략은 말 그대로 트랜잭션 범위와 영속성 컨텍스트의 생존 범위가 같다는 뜻이다.
  * 트랜잭션을 시작할 때 영속성 컨텍스트를 생성하고 트랜잭션이 끝날 때 영속성 컨텍스트를 종료한다.
  * 그리고 같은 트랜잭션 안에서는 항상 같은 영속성 컨텍스트에 접근한다.

* 스프링 프레임워크를 사용하면 보통 비즈니스 로직을 시작하는 서비스 계층에 @Transactional 어노테이션을 선언해서 트랜잭션을 시작한다. 외부에서는 단순히 서비스 계층의 메소드를 호출하는 것처럼 보이지만 이 어노테이션이 있으면 호출한 메소드를 실행하기 직전에 스프링의 트랜잭션 AOP가 먼저 동작한다.

* 스프링 트랜잭션 AOP는 대상 메소드를 호출하기 직전에 트랜잭션을 시작하고, 대상 메소드가 정상 종료되면 트랜잭션을 커밋하면서 종료한다. 

  * 이때 중요한 일이 일어나는데 트랜잭션을 커밋하면 JPA는 먼저 영속성 컨텍스트를 플러시해서 변경 내용을 DB에 반영한 후에, DB 트랜잭션을 커밋한다.
  * 따라서, 영속성 컨텍스트의 변경 내용이 DB에 정상 반영 된다.
  * 만약 예외가 발생하면 트랜잭션을 롤백하고 종료하는데, 이때는 플러시를 호출하지 않는다.

* **트랜잭션이 같으면 같은 영속성 컨텍스트를 사용한다.**

  * 만약에 MemberService 클래스의 findMember에서 FirstRepository와 SecondRepository의 메서드를 사용한다고 가정하자.

  * 트랜잭션 범위의 영속성 컨텍스트 전략은 다양한 위치에서 엔티티 매니저를 주입받아 사용해도 트랜잭션이 같으면 항상 같은 영속성 컨텍스트를 사용한다.

  * 아래의 코드에서 FirstRepo와 SecondRepo에서 엔티티매니저가 달라도 같은 트랜잭션 안에 있으므로 동일한 영속성 컨텍스트를 사용한다.

    ```java
    // MemberService.java
    
    @Service
    public MemberService {
      
      //엔티티 매니저 주입
      @PersistenceContext
      EntityManager em;
      
      @Autowired
      FirstRepository firstRepository;
      
      @Autowired
      SecondRepository secondRepository;
      
      @Transactional
      public void findMember() {
        firstRepository.hello();
        
        Member member = secondRepository.findMember();
        
        return member;
      }
    }
    ```

    ```java
    @Repository
    class FirstRepository {
      
      // 엔티티 매니저 주입
      @PersistenceContext
      EntityManager em;
      
      public void hello() {
        em.xxx() // 영속성 컨텍스트 접근
      }
    }
    ```

    ```java
    @Repository
    class SecondRepository {
      
      // 엔티티 매니저 주입
      @PersistenceContext
      EntityManager em;
      
      public Member findMember() {
        em.find(Member.class, "id1") // 영속성 컨텍스트 접근
      }
    }
    ```

* **트랜잭션이 다르면 다른 영속성 컨텍스트를 사용한다.**

  * 여러 스레드에서 동시에 요청이 와서 같은 엔티티 매니저를 사용해도 트랜잭션에 따라 접근하는 영속성 컨텍스트가 다르다.
  * 엔티티 매니저는 여러 스레드가 동시에 접근하면 동시성 문제가 발생한다.
    * 그래서 JPA는 스레드가 하나 생성될 때 마다(매 요청마다) EntityManagerFactory에서 EntityManager를 생성한다.
  * 스프링 컨테이너는 스레드마다 각각 다른 트랜잭션을 할당한다.
    * 결국 스프링에서는 매요청마다 하나의 엔티티 매니저와 하나의 트랜잭션을 사용함.
  * 두개의 스레드가 같은 엔티티 매니저를 사용하더라도, 트랜잭션이 다르므로 영속성 컨텍스트가 꼬일 일이 없다!!!
  * 따라서, **같은 엔티티 매니저를 호출해도 접근하는 영속성 컨텍스트가 다르므로 멀티스레드 상황에 안전하다.**
  * **실제로 어떻게 스프링은 엔티티 매니저의 thread-safety를 보장할까?**
    * @PersistenceContext 애노테이션으로 주입받은 EntityManager를 Proxy로 감싼다.
    * 이후 EntityManager 호출 시 마다 Proxy를 통해 EntityManager를 생성 하여 Thread-Safety를 보장 한다.
    * 이런것들을 스프링이 해주기 때문에, 비즈니스로직에 집중할 수 있다.
    * https://lng1982.tistory.com/276
    * [https://medium.com/@SlackBeck/spring-container%EB%8A%94-jpa-entitymanager%EC%9D%98-thread-safety%EB%A5%BC-%EC%96%B4%EB%96%BB%EA%B2%8C-%EB%B3%B4%EC%9E%A5%ED%95%A0%EA%B9%8C-1650473eeb64](https://medium.com/@SlackBeck/spring-container는-jpa-entitymanager의-thread-safety를-어떻게-보장할까-1650473eeb64)