### AWS Lambda

* 동작원리

  * 코드 등록
    * node.js, java, python  등 다양한 언어
    * 배포 옵션
      * 콘솔 편집기 사용
      * 코드를 zip으로 묶어 람다 또는 S3로 전송
      * Cloud9 IDE
      * CodeDeploy로 Canary 배포
  * 트리거
    * 이벤트 트리거
    * 여러 aws 서비스들과 통합
  * 실행
  * 사용요금
    * 람다의 실행시간 만큼만 과금
    * 할당된 메모리에 비례하여 CPU 및 네트웍 자원 할당
  * 모니터링 및 로깅
    * AWS CloudWatch
    * AWS CloudWatch Log
    * AWS X-Ray

### AWS Lambda의 이벤트 동작 방식

1. 이벤트 소스
   * 데이터 변화
     * (AWS) S3, DynamoDB, Kinesis, Congnito
   * 직접 또는 EndPoint로 호출
     * (AWS) Alexa, API Gateway, IoT
   * 리소스 상태 변화
     * (AWS) CloudFormation, CloudTrail, CodeCommit, CloudWatch
   * CloudWatch 알람/이벤트
     * (AWS) SES, SNS
   * Cron 주기별로 실행

2. 람다 함수
   * 실행 모델
     * 동기모델
       * ex) API Gateway를 통해 동기 처리
     * 비동기모델
     * Stream-based 모델
       * ex) Kinesis나 DynamoDB를 통해 들어오는 데이터 기반으로 Stream-based 처리
3. 외부 서비스 호출

### AWS Step Functions

* 워크 플로우 오케스트레이션 서비스
* 람다를 효율적으로 사용하기 위해 람다 Function들 간의 플로우를 설정할 수 있음

### AWS API Gateway

* 기존의 어려움
  * 다양한 API 버전 및 스테이지 관리 어려움
  * 타사 개발자들의 접근 모니터링에 시간 소요
  * 접근 권한 관리 어려움
  * 트래픽 증감에 따른 운영 부담
* 개요
  * 다양한 API 버전 및 스테이지 제공
  * 타사 개발자에 API 키 생성 및 배포
  * API 접근 권한 관리를 위한 AWS 서명 버전 4 활용
* 장점
  * 응답 저장을 위한 관리형 캐시
  * CloudFront를 통한 응답속도 개선 및 DDos 보호
  * ios, js등을 위한 SDK 생성
  * API 정의를 위한 Swagger 지원
  * Req/Res 데이터 변환

### AWS SAM(Serverless Application Model)

* 서버리스 앱의 컨텐츠를 기술하기 위한 공통 언어

### Lambda를 위한 CI&CD

* 코드가 커밋되면 파이프라인 실행
* 코드 레벨 빌드, 테스트, 검증
* Function 및 end-to-end 레벨의 연동 테스트
* **AWS CodePipeline**
  * 소스
    * AWS CodeCommit
  * 빌드
    * AWS CodeBuild
  * 테스트
    * AWS X-Ray
  * 프로덕션
    * AWS CodeDeploy

### Hands On Lab

* http://wiki.studydev.com/pages/viewpage.action?pageId=26903060