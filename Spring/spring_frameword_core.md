# 스프링 프레임워크 핵심 기술

> '기본에 충실하라'를 지키려고 노력중

## 스프링 소개

* 소규모 애플리케잉션 또는 기업용 애플리케이션을 자바로 개발하는데 있어 유용하고 편리한 기능을 제공하는 프레임워크
* 스프링 역사
    * 2003년 등장(개발은 이미 그 이전부터)
        * 등장시 Java EE  표준과 싸우는 것처럼 보였지만 실제로는 JEE 표준 스펙 구현 모음체(+알파)
        * 최근까지 주로 서블릿 기반 애플리케이션을 만들 때 사용해 옴
        * 스프링 5부터는 WebFlux 지원으로 서블릿 기반이 아닌 서버 애플리케이션도 개발할 수 있게 됨
* 디자인 철학
    * 모든 선택은 개발자의 몫(ex. 스프링이 특정 영속화 기술을 강요하지 않음)
    * 다양한 관점을 지향한다(유연성)
    * 하위 호환성을 지킨다(노력)
    * API를 신중하게 설계 한다(공들인다)
    * 높은 수준의 코드를 지향한다(자랑)

## IoC 컨테이너와 빈

### 1. 스프링 IoC 컨테이너와 빈

* **Inversion of Control**
    * 의존 관계 주입(Dependency Injection)이라고도 하며, 어떤 객체가 사용하는 의존 객체를 직접 만들어 사용하는게 아니라, 주입 받아 사용하는 방법을 말함.
* **스프링 IoC 컨테이너**
    * BeanFactory
        * 스프링 프레임워크의 중요한 인터페이스
    * 애플리케이션 컴포넌트의 중앙 저장소
    * 빈 설정 소스로부터 빈 정의를 읽어들이고, 빈을 구성하고 제공한다.
* **Bean - 빈**
    * 스프링 IoC 컨테이너가 관리 하는 객체
    * 장점
        * 의존성 관리
            * 의존성 관리를 할 수 없다고 생각해보자
            * 예를 들어, 서비스 객체 내에서 Repository 의존성을 직접 만들어서 사용하 는 경우
            * 서비스 단의 단위 테스트를 독립적으로 작성하기가 힘들다.
            * 의존성을 주입 받을 수 있기 때문에 객체(Repository)를 Mocking 해서 얼마든지 단위 테스트(Service)를 할 수 있다.
        * 스코프
            * 싱글톤: 하나
                * 빈의 기본 스코프 설정은 싱글톤
                * 메모리 관리 등 성능 최적화 이점
            * 프로토타입: 매번 다른 객체 생성
        * 라이프 사이클 인터페이스
            * @PostConstruct 등
* **ApplicationContext**
    * ApplicationContext는 spring-context 모듈 안에 위치함
    * spring-beans 모듈의 BeanFactory를 상속받고, 그외 다양한 기능을 가지는 인터페이스도 추가로 상속받은 인터페이스
        * ListableBeanFactory
        * HierarchicalBeanFactory
    * EnvironmentCapable
        * spring-core 모듈
    * MessageSource
        * spring-context 모듈
        * 메세지 소스 처리 기능(i18n)
    * ApplicationEventPublisher
        * spring-context 모듈
        * 이벤트 발행 기능
    * ResourcePatternResolver
        * spring-core 모듈
        * ResourceLoader
        * 리소스 로딩 기능 등

## Reference

* 스프링 프레임워크 핵심 기술 - 백기선