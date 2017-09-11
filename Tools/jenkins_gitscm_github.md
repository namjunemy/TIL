# 젠킨스와 Github 연동하기 

젠킨스와 Github을 연동하고, 추후에 Github Webhook을 이용한 자동 빌드환경 구축을 위해서 Github web에서 Personal access token을 발급 받고, 웹 브라우저 상에서 빌드를 진행하고 결과를 확인 할 수 있다.

*자세한 설명이 더 도움이 될 것 같아서 블로그에 따로 포스팅 했습니다. 화면 설명과 같이 자세한 설명을 보려면 [블로그 포스팅](http://ict-nroo.tistory.com/35)을 참조하는 것을 추천합니다 :)*



### Github 계정의 Jenkins Access Token 발급 받기

* Github 로그인 후 사용자 settings 탭 진입
* 왼쪽의 메뉴 중 Developer settings의 Personal access tokens 진입
* Pensonal access token의 generate new token 클릭
* Token description에 토큰 이름, Select scopes에서 repo와 admin:repo_hook 체크 후 생성
* 생성된 access token을 복사한다. Jenkins 시스템 설정에서 쓰인다.



### 젠킨스 관리 시스템 설정

* 젠킨스 관리 -> 시스템 설정 진입
* 젠킨스 location ip가 localhost 또는 127.0.0.1로 설정 되어 있다면 외부 접근(Github)을 위해 자신의 ip로 변경한다. `http://자신의 ip:8080/jenkins`
* Github 설정 탭에서 Manage hooks 체크 후, Credentials Add -> Jenkins 진입
* `Kind` : Secret text, `Secret` : Github에서 복사한 Personal access token, `ID` : Github 아이디 입력 후 Add로 Credentials 추가
* Test Connection을 통해서 Credentials가 잘 추가 됐는지 확인한다.
* 설정 완료



### 젠킨스 프로젝트 만들기

* 새 아이템 만들기 -> Freestyle project 선택 후 프로젝트 생성



### 젠킨스 프로젝트 구성

새 프로젝트를 생성한 후 시스템 구성 탭으로 진입한다.

* Github project 설정
  * Github repository에서 https 주소 복사 후 Jenkins 설정 탭의 Github project 탭에 url을 추가한다.
* 소스 코드 관리 설정
  * 소스 코드 관리 메뉴에서 Git체크 후, Github저장소 url 입력
  * Kind를 Username with password로 설정한 후 Username, Password, ID 입력(젠킨스)
* 빌드 도구 설정
  * 빌드 툴로 Gradle을 선택했기 때문에, Build 탭에서 Invoke Gradle script를 통해 Gradle Build에 대한 설정을 한다. Gradle version 설정 후, Tasks는 clean, build로 설정해준다.