# API 개발과 성능 최적화

> [실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94/dashboard) 핵심 정리

![](https://github.com/namjunemy/TIL/blob/master/Jpa/img/relation_01.PNG?raw=true)

## 지연 로딩과 조회 성능 최적화

* 주문 + 배송정보 + 회원을 조회하는 API를 만든다
* 지연 로딩 때문에 발생하는 성능 문제를 단계적으로 해결해보자

### 페치 조인 최적화

* order 목록을 조회하는 상황에서 Member정보와 Delevery정보까지 함께 한방에 가져오고 싶다.
* Order 엔티티에 Member, Delivery는 페치 타입을 Lazy로 설정한 상태에서 쿼리가 여러방 나가는 **JPA 1 + N 문제를 해결**하고 싶을 때 페치 조인 최적화를 활용한다.
* 쿼리 1방에 해결.

### DTO 바로 조회

* 엔티티 전체를 조회하지 않고, DTO로 바로 조회하는 방법
* 엔티티 전체를 조회하는 것과 DTO로 바로 조회하는 것은 **트레이드 오프**가 있다.
* new 명령어를 사용해서 JPQL의 결과를 DTO로 즉시 변환한다.
* SELECT 절에서 원하는 데이터를 직접 선택하므로 DB -> 애플리케이션 네트웍 용량 최적화(하지만, 요즘 네트워크 성능이 좋아서 생각보다 미비하다.)
* 엔티티 전체를 조회하면 재사용성을 확보할 수 있지만, DTO바로 조회는 딱 그 DTO에만 사용 가능하다.
* API 스펙에 맞춘 코드가 repository에 들어가는 단점이 있다. repository가 화면을 의존하는 결과를 낳는다.
* 정말 DTO바로 조회가 필요하다면, 기존의 Repository는 순수한 엔티티를 조회하는 메서드들만 남기고 새로운 SimpleQueryRepository를 만들어서 DTO와 함께 패키지를 분리한다. 유지보수 하기 좋다.

### 쿼리 방식 선택 권장 순서

1. 우선 엔티티를 DTO로 변환하는 방법을 선택한다.(유지보수 관점에서 좋다. )
2. 필요하면 페치 조인으로 성능 최적화 한다. -> 대부분 성능 이슈가 해결 된다.
3. 그래도 안되면 DTO로 직접 조회하는 방법을 사용한다.
4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링JDBC Template을 사용해서 SQL을 직접 사용한다.