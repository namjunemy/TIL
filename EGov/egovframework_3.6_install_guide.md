# eGovFramework 3.6 Install Guide

>  전자정부 프레임워크 3.6 버전 개발환경을 세팅하기 위한 설치 가이드
>
>  [http://www.egovframe.go.kr/](http://www.egovframe.go.kr/)'
>
>  환경 : Windows 10 / Jdk 1.8 / tomcat 8.0.X / maven(추가 설치)

  

#### 개발환경설치

* eclipse 기반의 전자정부 표준 프레임워크의 구현도구(implementation tool) 설치를 참조하여 설치한다.
* 설치 링크 :  [http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:dev3.6:clntinstall](http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:dev3.6:clntinstall)
* Step 1의 개발환경 설치에서 개발자 개발환경 다운로드만 진행하고, 더이상 진행하지 않는다.
* 플러그인 업데이트와 maven repo설정을 진행 한 뒤에 sample project를 실행시킬 것이다.

#### 플러그인 업데이트

* 원하는 기능만을 선택하여 개발환경을 구성 할 수 있는 기능을 제공한다.
* [http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:dev2:imp:editor:customize_development_tool#사용법](http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:dev2:imp:editor:customize_development_tool#사용법)
* 위의 링크로 이동하여 필요한 플러그인의 업데이트를 진행한다.

#### Maven 환경설정

* [http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:dev3.6:gettingstarted#step_3._자세히_들여다_보기](http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:dev3.6:gettingstarted#step_3._자세히_들여다_보기)
* 위의 링크로 이동하여 Step1. 개발환경 설치 탭에서 Maven 환경설정으로 이동하여 순차적으로 Maven 환경설정한다.
* 특히, settings.xml 설정을 잘 했는지 확인한다.

#### Sample 프로젝트 실행

* [http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:dev3.6:clntinstall](http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:dev3.6:clntinstall)
* 위의 링크로 이동하여 개발 환경 설치 가이드로 이동하여, 개발자 개발환경 다운로드 탭의 다음 순서를 순차적으로 실행하면 tomcat 위에서 구동되는 웹 프로젝트를 볼 수 있다.