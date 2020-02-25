# Spring 5 recipes

> 스프링 5 레시피 - 마틴 데니엄, 다니엘 루비오, 조시 롱 지음 참조.
>
> 스프링 애플리케이션 개발에 유용한 161가지 문제 해결 기법

# 스프링 코어

## 2-1. 자바로 POJO 구성하기

### 구성 클래스에서 @Configuration과 @Bean을 붙여 POJO 생성

```java
@Configuration
public class SequesceGeneratorConfiguration {
  
  @Bean
  public SequenceGenerator sequenceGenerator() {
    
  }
}
```

* @Bean(name="myBean1") 과 같이 name을 설정하면 빈 생성시 메서드 명은 무시

### IoC 컨테이너를 초기화하여 애너테이션 스캐닝하기

* 애너테이션을 붙인 자바 클래스를 스캐닝하려면 우선 IoC 컨테이너를 인스턴스화 해야 한다. 그래야 스프링이 나중에 @Configuration과 @Bean등을 발견하고 IoC 컨테이너에서 빈 인스턴스를 가져올 수 있다.

* 스피링은 기본 구현체인 Bean Factory와 이와 호환되는 고급 구현체인 Application Context, 두 가지 IoC 컨테이너를 제공한다.

* 애플리케이션 컨텍스트는 기본 기능에 충실하면서도 빈 팩토리보다 발전된 기능을 지니고 있으므로 리소스에 제약을 받는 상황이 아니면 가급적 애플리케이션 컨텍스트를 사용하는게 좋다. 

* BeanFactory와 ApplicationContext는 각각 빈 팩토리와 애플리케이션 컨텍스트에 접근하는 인터페이스이다. ApplicationContext는 BeanFactory의 하위 인터페이스여서 호환성이 보장된다.

* ApplicationContext는 인터네이스이기 때문에 사용하려면 구현체가 필요하다. 스프링은 몇가지 애플리케이션 컨텍스트 구현체를 마련해 놓았다.

* 그중 가장 최근 작품이면서 유연한 AnnotationConfigApplicationContext를 권장한다.

  ```java
  ApplicationContext context = new AnnotationConfigApplicationContext(SequeceGeneratorConfiguration.class);
  ```

* 애플리케이션 컨텍스트를 인스턴스화 하면, 이후에 객체 레퍼런스(코드에서 context)는 POJO 인스턴스 또는 빈에 엑세스하는 창구 역할을 한다.

### IoC 컨테이너에서 POJO 인스턴스/빈 가져오기

* 구성 클래스에 선언된 빈을 빈 팩토리 또는 애플리케이션 컨텍스트에서 가져오려면 유일한 빈 이름을 getBean() 메서드의 인수로 호출해야 한다. 리턴타입은 Object 이므로 실제 타입에 맞게 캐스팅이 필요하다

  ```java
  SequenceGenerator generator = (SequenceGenerator) context.getBean("sequenceGenerator");
  ```

* 캐스팅 하지 않으려면, 빈 클래스명 지정.

  ```java
  SequenceGenerator generator = context.getBean("sequenceGenerator", SequenceGenerator.class);
  
  // 빈이 하나뿐이라면 빈의 이름 생략 가능
  SequenceGenerator generator = context.getBean(SequenceGenerator.class);
  ```

* 이런식으로 POJO 인스턴스/빈을 스프링 외부에서 생성자를 이용해 여느 객체처럼 사용할 수 있다.

  ```java
  public class Main {
    
    public static void main(String[] args) {
      ApplicationContext context = 
        new AnnotationConfigApplicationContext(SequenceGeneratorConfiguration.class);
      
      SequenceGenerator generator = context.getBean(SequesceGenerator.class);
      
      System.out.println(generator.getSequence());
      System.out.println(generator.getSequence());
    }
  }
  
  // return example
  3010000A
  3010001A
  ```

### POJO 클래스에 @Component를 붙여 빈 생성

* 클래스에 @Component("sequesceDao")를 붙이면 스프링은 이 클래스를 이용해서 POJO를 생성한다.

* @Component에 넣은 값은 빈 인스턴스의 ID로 설정하며, 값이 없으면 소문자로 시작하는 비 규격 클래스명을 빈 이름으로 기본 할당한다. 여기서는 sequenceDaoImpl

  ```java
  @Component("sequenceDao")
  public class SequenceDaoImpl implements SequenceDao {
    
    ...
  }
  ```

* @Component는 스프링이 발견할 수 있게 POJO에 붙이는 범용 에너테이션이다.

* 스프링에는 Persistence, Service, Presentation 각 영속화, 서비스, 표현 세가지 계층이 있는데,

* @Repository, @Service, @Controller가 이 세 계층을 가리키는 에너테이션이다.

* **POJO의 쓰임새가 명확하지 않을 때는 그냥 @Component를 붙여도 되지만 특정 용도에 맞는 부가 혜택을 누리려면 구체적으로 명시하는 편이 좋다.**

  * **예를 들면, @Repository는 발생한 예외를 DataAccessException으로 감싸 던지기므로 디버깅시 유리하다.**

### 애너테이션을 스캐닝하는 필터로 IoC 컨테이너 초기화하기

* 기본적으로 스프링은 @Configuration, @Bean, @Component, @Repository, @Service, @Controller가 달린 클래스를 모두 감지한다.

* 이때 하나 이상의 포함/제외 필터를 적용해서 스캐닝 과정을 커스터마이징 할 수 있다.

* 스프링에서 지원하는 필터 표현식은 네 종류이다.

  * annotation, assignable은 각각 애너테이션 타입 및 클래스/인터페이스를 지정하고,
  * regex, aspectj는 각각 정규표현식과 AspectJ 포인트컷 표현식으로 클래스를 매치하는 용도로 쓰인다.

* use-default-filters 속성으로 기본 필터를 해제할 수도 있다.

* 다음과 같이 선언하면 io.namjune.speingrecipes.sequence 패키지에 속한 클래스 중 이름이 Dao나 Service가 포함된 것들은 모두 넣고, @Controller를 붙인 클래스는 스캐닝 대상에서 제외한다.

  ```java
  @ComponentScan(
  	includeFilters = {
      @ComponentScan.Filter(
      	type = FilterType.REGEX,
        pattern = {"io.namjune.speingrecipes.sequence.*Dao",
                   "io.namjune.speingrecipes.sequence.*Service"})
    },
    includeFilters = {
      @ComponentScan.Filter(
      	type = FilterType.ANNOTATION
        classes = {org.springframework.stereotype.Controller.class}})
    }
  )
  ```

