# MySQL 사용자 추가/삭제, 권한 제어

> 관리자 권한 명령 프롬프트 실행
>
> MySQL 5.7 버전 이상

####접속하기

* `> mysql -u root -p`

#### 사용자 확인하기

mysql database 선택하고 host, user 정보 확인

* `mysql> user mysql;` 
* `mysql> select host,user,authentication_string from user;`

#### 계정 외부 접속 허용 

host 종류는 'localhost'와 '%'가 있다. 기본적으로 MySQL을 설치하면 로컬(localhost)에서만 접속이 가능하고 외부에서는 접속이 불가능하게 되어 있다.

따라서, root계정의 외부접속을 허용하려면 다음과 같은 쿼리를 날리면 된다.

* ` mysql> inster into mysql.user (host, user, authentication_string, ssl_cipher, x509_issuer, x509_subject) values ('%','사용자명', password('비밀번호'),'','','');`
* `mysql> grant all privileges on *.* to 'root'@'%';`
* `mysql> flush privileges;`

#### 사용자 추가

외부 접속 허용과 동일한 루틴으로 사용자 추가도 진행 할 수 있다.

* ` mysql> inster into mysql.user (host, user, authentication_string, ssl_cipher, x509_issuer, x509_subject) values ('% or localhost','사용자명', password('비밀번호'),'','','');`
* `mysql> grant all privileges on *.* to 'root'@'% or localhost';`
* `mysql> flush privileges;`

#### 사용자 제거

* `mysql> drop user '사용자 ID';` 또는
* `mysql> delete from user where user = '사용자 ID';`

#### 사용자에게 데이터베이스 사용권한 부여

* `mysql> grant all privileges on dbname.table to userid@host identified by 'password';`
* `mysql> grant select, insert, update on dbname.table to userid@host identified by 'password';`
* `mysql> grant select, insert, update on dbname.table to userid@'192.169.%' identified by 'password';`
  * host가 192.168.X.X로 시작되는 모든 IP의 원격 접속을 허용
* dbname.table 대신 dbname.* 은 해당 database의 모든 table의 접근을 허용한다. \*.\*은 모든 접근을 가능하게 한다.

#### 변경된 권한 적용

* `mysql> flush privileges;`

#### 권한 삭제

* `mysql> revoke all on dbname.table from username@host;`

#### 권한 확인

* `mysql> show grants for userid@host;`
* `mysql> show grants for 'njkim'@'%';`