# Gradle 설치 및 프로젝트 생성 

> OS : Windows10
>
> IDE : Eclipse Oxygen Release (4.7.0)
>
> Gradle : 4.1
>
> Git : 2.14.1



### 사전 작업

* Git 설치(Git Bash 사용, Windows PowerShell 사용해도 무방)



### Gradle 설치

* [https://gradle.org/releases](https://gradle.org/releases) 방문 Gradle latest version 다운
* 설치를 원하는 위치에서 압축 해제
* 윈도우 환경변수 설정
  * 제어판 -> 시스템 -> 고급 시스템 설정 -> 고급 탭의 "환경 변수(N)"로 진입
  * 시스템 변수 -> 새로 만들기 -> 변수 이름 : GRADLE_HOME -> 변수 값 : gradle root 위치 -> 생성
  * 시스템 변수(S)에서 Path를 찾아 클릭하고 "새로 만들기(N)" 버튼 클릭 후 **%GRADLE_HOME%\bin** 경로를 추가한다.
* gradle 설치 확인
  * git bash 새 창 열기
  * gradle -v -> 설치 된 버전 확인


### Gradle 프로젝트 만들기

* 파일 탐색기 or git bash cli 에서 파일구조 만들기

  > 파일구조는 다음과 같은 표준을 따르고 있다.
  >
  > * 프로젝트 루트(프로젝트명)
  >   * src
  >     * main
  >       * java
  >       * resources
  >       * webapp
  >     * test
  >       * java
  >       * resources

* 프로젝트 루트에 build.gradle 파일 생성

  > 다음 내용 입력 후 저장
  >
  > vi or 메모장(모든파일 -> build.gradle)
  >
  >  
  >
  > apply plugin: 'java'
  >
  > apply plugin: 'eclipse'

* 프로젝트 루트에서 git bash open 후 ls 명령 입력시

  * build.gradle과 src디렉토리가 존재하는 상황

* `$ gradle eclipse` 명령 입력

  > 아래와 같은 내용 출력시 빌드 성공
  >
  > Starting a Gradle Daemon (subsequent builds will be faster)
  >
  > :eclipseClasspath
  >
  > :eclipseJdt
  >
  > :eclipseProject
  >
  > :eclipse
  >
  > ​
  >
  > BUILD SUCCESSFUL in 8s
  >
  > 3 actionable tasks: 3 executed

* 새로 만든 프로젝트는 eclipse ide에서 import 가능한 프로젝트가 됨

