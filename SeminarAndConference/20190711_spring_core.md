# 하루에 정리하는 Spring Framework Core

> 2019-07-11
>
> NHNEntertainment 김병부 책임

## 1

- Spring container == ApplicationContext 로 보자

## 2

- Bean의 멤버변수 선언할 때, singleton은 JVM 안에서 singleton이 아니다. ApplicationContext 안에서의 singleton이다.
- 그리고, @Bean이 Singleton 인데. 멤버변수가 HashMap이다. 망했다.
    - 멀티 스레드에서 스레드 세이프를 보장 해줘야 안전하다. ConcurrentHashMap을 쓰자.

## 3

* @Configuration, @Profile("default")는 디폴트 프로파일로 설정
* PSA

## 4

* Spring AOP
    * **weaving의 대상이 반드시 Spring Container에 의해서 관리되는 Spring Bean이어야 한다.**
    * 즉, @Cachable, @Transactional을 @Service, @Controller 없이 쓸 수 없다는 이야기이다.
    * Spring Bean 이라도 private 이면 Spring AOP 를 쓸 수 없다.

* Annotation을 활용한 AOP
    * 포인트 컷을 메서드 이름이나 패키지 이름으로 짜는 것 보다.
    * 어노테이션을 바탕으로 AOP를 짜는 것을 추천한다.