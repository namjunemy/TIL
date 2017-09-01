# MySQL Installation guide

다음 가이드는 MySQL 5.7.18 이후 버전에 적용 가능한 설치 가이드 입니다.

설치 환경 : Windows 10 Pro / Mysql-5.7.19



* Download Mysql Community Server [다운로드 사이트](https://dev.mysql.com/downloads/mysql/)에 접속한다.
* OS와 시스템 종류를 선택하고 ZIP Archive 파일을 다운받는다.
* Oracle 계정으로 로그인 해야한다.
* ZIP파일 압축을 해제하고, 폴더명을 mysql로 변경한다.
* 제어판 -> 시스템 -> 고급 시스템 설정 -> 고급 탭의 "환경 변수(N)"로 진입한다.
* 시스템 변수(S)에서 Path를 찾아 클릭하고 "새로 만들기(N)" 버튼 클릭 후 D:\database\mysql\bin 경로를 추가한다.
* 이 때, mysql 앞 까지는 본인이 설치한 경로이다.
* D:\database\mysql\data 폴더를 생성한다.(DB 데이터가 저장되는 경로이다.)
* 명령 프롬프트를 관리자 권한(우클릭)으로 실행한다.
* 실행 후 `mysql.exe --initialize` 입력
* D:\database\mysql\data 폴더에 `DESKTOP-*.err` 파일을 열고 `Ctrl + F`를 사용하여 'temporary password'가 포함된 문장을 찾는다.
* 2017-09-01T05:37:32.561784Z 1 [Note] A temporary password is generated for root@localhost: `7w;Aw/_>9tbR` 이런식으로 패스워드가 보인다.
* 잘 복사하고, 명령 프롬프트(관리자 권한)에서 `D:\database\mysql\bin\mysql.exe --install` 을 입력한다.
* 'Service successfully installed'라는 리턴을 받는다면,
* 제어판 - 관리도구 - 서비스 - MySQL을 시작시킨다.
* 명령 프롬프트(관리자)에서 로그인을 위해 `mysql -u root -p` 입력 후, Enter Password: 라고 나오면 위에서 복사했던 패스워드를 입력한다.
* 정상적으로 접속이 됐다면 콘솔이 mysql>로 바뀐 것을 확인 할 수 있다.
* `set password = password('1111');` 입력을 통해서 root계정의 패스워드를 원하는 것으로 바꿔준다.
* `quit` 명령을 통해 접속 해제 한 후, 다시 로그인 하면 바뀐 패스워드로 로그인이 가능하다.
* `show databases;` 명령을 통해 데이터베이스 리스트를 확인 할 수 있다.