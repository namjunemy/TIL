# 젠킨스 빌드 환경 구축

젠킨스와 버전 관리 시스템 그리고 빌드 툴을 연동하여 젠킨스를 통한 빌드가 이루어 질 수 있도록 환경을 구축하는 실습 예제이다.

*자세한 설명이 더 도움이 될 것 같아서 블로그에 따로 포스팅 했습니다. 화면 설명과 같이 자세한 설명을 보려면 [블로그 포스팅](http://ict-nroo.tistory.com/34)을 참조하는 것을 추천합니다 :)*



>  Windows 10 / JDK 1.8 / Tomcat 8.5.20 / Git 2.14.1 / Jenkins 2.60.3 / Gradle 4.1 / github



### 젠킨스 플러그인 설치

실습에서 필요한 플러그인은 Git / Github / Gradle Plugin이고, 각 플러그 인을 검색하여 설치를 진행한다.

* 메인 페이지 왼쪽 사이드 메뉴의 `Jenkins 관리` 탭으로 이동한다.
* `Jenkins 관리`  메뉴의 플러그인 관리 탭으로 이동한다.
* `설치 가능`  탭으로 이동하여 설치하고자 하는 플러그인을 체크 한 후 설치한다. 



###  Git 설치

* [https://git-scm.com/](https://git-scm.com/) 링크로 이동한다.
* 메인화면 오르쪽에 Downloads for Windows를 통해 설치한다.
* 설치 위치를 지정하고 default 설정으로 설치한다.
* 설치 위치로 이동하여 git bash를 실행하고 git --version 명령을 통해 설치 된 git의 버전을 확인한다.



### Build Tool 설치

* [https://gradle.org/releases/](https://gradle.org/releases/) 링크로 이동한다.
* Gradle latest version을 다운 받는다.
* 설치를 원하는 위치에서 압축을 해제한다.
* 윈도우 환경 변수를 설정한다
  * 제어판 -> 시스템 -> 고급 시스템 설정 -> 고급 탭의 "환경 변수 (N)"로 진입
  * 시스템 변수 -> 새로 만들기 -> 변수 이름 : GRADLE_HOME -> 변수 값 : GRADLE root 위치
  * "시스템 변수(S)"에서 Path를 클릭하고 "새로 만들기(N)" 버튼 클릭 후 "%GRADLE_HOME%\bin" 경로 추가
* Git bash 쉘을 열고 gradle -v 명령을 통해 설치 된 버전을 확인한다.



### 젠킨스 Tool Configuration 설정

젠킨스와 연동할 Tool의 Configuration을 설정한다. Tool들이 로컬 환경에 설치 되어 있다면 install automatically 메뉴에 체크 해제하고 로컬에 설치된 Path를 등록한다.

* `Jenkins 관리`  탭의 Global Tool Configuration 메뉴를 선택한다.
* JDK install automatically에 체크 해제 하고, JAVA_HOME의 Path를 입력한다. 제대로 입력하지 않으면 잘못된 Path라는 경고가 발생한다.
* Git install automatically에 체크 해제 하고, Git의 Path를 입력한다. 여기서는 git.exe까지 입력해야 한다.
* Gradle install automatically에 체크 해제 하고, GRADLE_HOME의 Path를 입력한다.
* Tool의 Path 설정이 끝났다면 설정을 저장한다.


