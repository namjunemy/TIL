# 젠킨스 설치 및 구동

젠킨스는 자바로 개발되었으며 서블릿 컨테이너 위에서 구동된다. 따라서 WAS인 Toacat, Jetty 등 서블릿 컨테이너를 설치하고 이 위에서 젠킨스 war파일로 구동한다.

각 OS별로 별도의 설치파일을 제공하고 있으나, OS에 제한적이지 않고 WAS에 배포함으로써 구동할 수 있도록 .war파일을 다운받았다.

*환경변수 설정에 관련 된 내용은 생략하겠습니다.* 

### JDK 설치 

* [http://www.oracle.com/technetwork/java/javase/downloads/index.html](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 접속 후 JDK 설치
* windows 환경 변수 설정



### WAS 설치

* [http://tomcat.apache.org/](http://tomcat.apache.org/) 접속 후 Tomcat 설치
* 원하는 위치에 압축 해제



### Jenkins 설치

* [https://jenkins.io/](https://jenkins.io/) 접속 후 LTS 버전 탭 이동
* Download Jenkins X.XX.XX for Generic Java Package(.war) 파일 설치



### Jenkins 구동

* Tomcat의 설치 위치로 이동한다.


* webapps 폴더 아래 설치한 jenkins.war파일을 배포하고 Tomcat을 구동한다.
* Tomcat의 구동은 파일 탐색기 진입 후 C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Administrative Tools 이동 "서비스" 앱 실행 또는 시작메뉴 -> Windows 관리도구 -> 서비스 앱을 실행 한 후 Apache Tomcat 8.5 TomcatX 서비스를 시작 시킨다.
* 웹브라우저의 주소창에 localhost:8080/jenkins 또는 127.0.0.1/jenkins 또는 자신의ip:8080/jenkins를 입력하고 접속하여 젠킨스가 정상적으로 구동 되는지 확인한다.
* 접속 후 Secret Key를 입력하는 화면이 나오면, key의 경로에서 Secret Key 값을 복사하여 입력한다.
* 키를 입력하고, lock이 풀리면 install suggested plugins를 클릭하여 기본 플러그인을 설치한다.
* 다음 순서로 관리자 계정을 만들면 젠킨스 메인 페이지 접속에 성공 할 수 있다.