# 우아한 스프링 배치

> 2019.09.26 우아한형제들 이동욱
>
> 기본편 - 활용편을 위한 기본 개념 & 오해 풀기
>
> 활용편 - 실제 업무에서의 스프링 배치

## 기본편

### Web vs Batch

* web
    * 실시간 처리 / 상대적인 속도 / QA 용이
* batch
    * 후속 처리 / 절대적인 속도 / QA 복잡
    * QA가 복잡하므로 테스트 코드를 꼭 짜자.

### Batch와 Quartz

* Quartz는 스케줄링 프레임워크이다. 
    * Ex) 매 시간 / 마지막 주 금요일에 실행 등
* 쿼츠는 스프링 배치의 보안제 역할이지 대체제가 아니다.
* 쿼츠로 코드를 짜면 스케줄링 어플리케이션이지 배치 어플리케이션이 아니다.
* 배치 애플리케이션이 필요한 상황
    * 일정 주기로 실행되어야 할 때
    * 실시간 처리가 어려운 대량의 데이터를 처리 할 때, 웹 API로 처리하기 힘든 대량의 데이터
* 배치는 대용량 데이터 처리가 절대적인 요구 사항이다.
* 스프링 배치에서는 모든 데이터를 메모리에 쌓는 방식이 기본이 아니다.
* 페이징 혹은 커서로 pageSize만큼만 읽어 오고, chunkSize 만큼만 커밋 한다.
* jpaRepository.findAll()로 진행하면 고생한다... 수억건 한번에 조회....

### Job / Step / Tasklet

* Tasklet과 reader, processor, writer 는 다른 개념이 아니다.
* reader, processor, writer 는 ChunkOrientedTasklet 이다. 청크 지향 태스크릿.
* JobParameter는 Enum, LocalDate, LocalDateTime을 지원 안한다.
    * 그래서 String 으로 받아서 매번 스텝에서 형변환 해서 쓴다. 매번. 개선할 수 없을까?
    * @Value의 특성을 사용하자.
    * **CreateJobParameter 클래스를 만들어서 배치 잡이 실행되는 시점에 (CreateJobParameter빈이 생성될 때) setter로 넣고 그 안에서 변환해 놓고,**
    * **잡, 스텝 안에서 getParameter()만 열어서 쓴다. (영상 참조)**
* 스프링 빈은 애플리케이션이 올라오는 시점에 생성된다.
* 배치에서는 잡이나 스텝이 실행될때 @JobScope, @StepScope가 선언되어있는 빈들이 Late Binding 된다.

### Late Binding

* 스프링 배치의 @JobScope, @StepScope LateBinding 특성을 이용해서 JobParameter 값에 따라 reader와 processor를 교체할 수 있다.

## 활용편

### 관리 도구들

* Cron
* Spring MVC + API Call
* Spring Batch Admin (Deprecated)
    * Spring Cloud Data Flow로 이전하라.
* Quartz + Admin
* **CI Tools (Jenkins / Teamcity 등)**

### 젠킨스의 장점

* Integration(Slack, Email) 연동이 잘된다
* 실행 이력 / 로그 관리 / DashBoard
* 다양한 실행 방법(Rest API / 스케줄링 / 수동 실행)
    * 크론과 쿼츠보다 장점.
* 계정별 권한 관리
* 파이프라인 구성
* Web UI + Script 둘다 사용 가능
* Plugin (엔서블, 깃헙, Logetries 등)
* 배치 또는 쿼츠 어드민은 이 기능들을 다 만들어야 된다 ...
* 젠킨스에서 배치 어플리케이션 실행하는법
    * java -jar  -DSpring.profiles.active=prod Application.jar --job.name=testBatch baseDate=${baseDate}
* 자바 8 까지는 G1GC가 기본이 아니다. 따라서 모든 배치 Job에 공통 코드로 G1GC와 jar 경로를 넣어줘야 된다.
* 젠킨스 공통 설정 관리에서 Global properties에 Environment variables에서 모든 옵션을 관리할 수 있다.
* 여기서 설정 다 만들고. 만들어진 $BATCH_JAR 를 모든 Job에서 사용할 수 있다.
    * java -jar \$BATCH_JAR --job.name=testBatch baseDate=​\${baseDate}

### 무중단 배포

* AWS 환경이 아닌 곳에서 SCP로 동일하게 수행할 수 있다.
* 배포 서버 젠킨스(모든 서버 접근 가능), 배치 서버 젠킨스(저장소 접근 가능)로 따로 서버를 둔다.
* 배포 젠킨스가 명령을 받으면 git pull 받아서 테스트 돌리고, 빌드해서 S3에 올리고
* 코드 디플로이를 통해서 배치 어플리케이션 실행한다.
* 마법의 명령어 $readlink
    * 링크파일을 실행하지 않고 원본 파일을 실행한다.
    * 배치 젠킨스에서 app.jar를 커맨드라인에서 실행시키는데 v1.jar가 실행된다.
    * 배포 젠킨스가 돌면서 v2.jar를 만들고 링크를 app.jar로 만든다.
    * 만약 v1.jar 배치어플리케이션이 돌고있더라도 영향없다.
* 모든 Job 마다 $readlink 넣어야 할까?
* \$readlink도 젠킨스에서 공통 변수로 등록해서 쓰자. \$BATCH_JAR 에 합친다.

### 파이프라인

* 젠킨스를 선택한 이유중 하나.
* 원본 잡 변경을 한 곳에서 하기 위해서 파이프 라인을 사용한다. 잡은 하나인데 파라미터만 다르거나 등.
* 순차적인 집계를 해야 할 때, 파이프라인 구성이 매우 도움을 줌.
* 실무 팁
    * **여러 작업이 순차적으로 실행이 필요할 때, 많은 사람들이 스텝으로 나눈다.**
    * **스텝으로 나누기 보다는 파이프라인을 우선 고려하자!**
    * 언제든지 개별적으로 수행 할 수 있도록 잡을 나누자.
    * 꼭 스텝으로 수행해야되? 라는 고민을 꼭 하자. 잡 : 스텝 -> 1 : 1 관계를 유지해서 독립 수행이 가능하도록 하자.

### 멱등성

* 연산을 여러번 적용하더라도 결과가 달라지지 않는 성질
* 애플리케이션 개발에서 멱등성이 깨지는 경우
    * 제어할 수 없는 코드를 직접 생성할 때
    * Scanner에 System.in 넣거나, LocalDateTime.now() 사용할 때.
    * 이 배치 어제 장애났는데, 어제 날짜로 한번 다시 돌려주세요.
    * 배치 잡 안에 LocalDateTime.now() 있는데?.....
    * 이런 경우가 멱등성이 깨지는 경우다.
    * 그래서 코드 수정하고 임시 배포해서 배치 돌리고 다시 롤백한다....
    * **멱등성을 지키려면 꼭 JobParameter로 입력 받아서 실행하자.**
    * LocalDateTime.now()를 쓰면 비즈니스 로직 짜기 너무 쉽다. 하지만 지양하자.
    * 젠킨스에서 어떻게 오늘 일자를 yyyy-MM-dd로 보내지?
    * 젠킨스에 **Date Parameter Plugin** 이 있다. Java8의 날짜 기능을 지원한다.
        * LocalDate.now().minusDays(1);

### 테스트 코드

* 웹 애플리케이션보다 배치 애플리케이션의 테스트 코드가 훨씬 중요하다.
* QA가 더 어렵기 때문이다.
* @ConditionalOnProperty를 사용해서 JobName이 환경변수로 넘어올때만 배치 Config를 활성화한다.(배치 빈으로 뜬다.)
* 테스트 할때는 환경변수 넣어야 되는데,
    * @TestPropertySource(properties = "job.name=customerCopyJob") 를 넣으면 된다.
* 이렇게 많이 사용하고 있었는데, 1200개 이상의 테스트 케이스를 돌리다보니까 엄청 복잡하다.
* 리더가 잘 동작하는지 보려고, 스프링 통합 테스트 한다.
* 100개가 넘기 시작하면서 전체 테스트 수행 속도가 너무 느려진다. 이상하다?
* 범인은 @ConditionalOnProperty 였다.
* **스프링 배치에서는 전체 테스트 수행시 Environment가 변경될 때마다 Spring Context를 재시작 한다.**
* 느려질 수 밖에 없었다.
* Environment가 변경되는 조건
    * 테스트에서 @MockBean / @SpyBean 사용할 때
    * 테스트에서 @TestPropertySource 로 환경변수 변경할 때
    * @ConditionalOnProperty 로 테스트마다 Config가 다를때
* 제거하자.
    * @ConditionalOnProperty
    * @TestPropertySource 둘다.
* 어떻게?
    * 모든 Config를 로딩한 뒤, 원하는 배치 Job Bean을 실행한다.
    * ApplicationContext를 주입받고, JobName으로 JobBean을 조회 후 테스트 대상에 포함시킨다.
    * 유틸리티형 컴포넌트 테스트 유틸을 만든다.
    * 이렇게 하니까 @ConditionalOnProperty, @TestPropertySource 둘다. 제거하고나서
    * 하나의 ApplicationContext로 모든 @SpringBootTest 케이스를 커버할 수 있다.
    * 여기서 @MockBean / @SpyBean 쓰면 새로 뜬다.

### JPA & Spring Batch

* JPA N+1 문제 fetch join으로 해결할 수 있다.
* 하지만 만병 통치약이 아니다.
* 하위 엔티티 2개 종류 이상에서 join fetch 사용시 MultiBagFetchException이 발생한다.
* Fetch join 말고 다른 해결방법이 default_batch_fetch_size 옵션을 줄 수 있다.
* 하위 엔티티를 로딩할때 지정된 숫자만큼 상위 엔티티 ID를 Where IN 절에 넣어서 조회한다.
* 값이 1000 이면 in 절에 id 1000개 넣고 조회한다.
* default_batch_fetch_size 만큼 in 절에 들어간다.
* 하위 엔티티가 1만개면 1000개씩 끊어서 10 + 1 번 쿼리가 나간다.
* 하지만 이 옵션은 JpaPagingItemReader에서는 작동하지 않는다.(HibernateCursorItemReader는 가능)
* 커서는 계속 커넥션을 물고 있어서, 대량의 데이터는 JpaPagingItemReader쓰는게 좋은데 현재 작동하지 않는 버그있어서 pr 날린 상태.

### Persist Writer

* 처음 데이터가 save가 될 때도 업데이트 쿼리가 항상 실행됨.
* 배치 4.2에서 개선 됐다.