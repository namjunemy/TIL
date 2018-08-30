# Redis on Windows

* 윈도우용 레디스 릴리즈는 깃헙에서 관리되고 있다.
  * https://github.com/MicrosoftArchive/redis/releases
  * 원하는 릴리즈 선택하고 .msi파일 다운로드
  * 설치위치 지정하고 설치, 설치시 환경변수 등록과 방화벽 예외 등록은 체크하기
* 설치 위치에서 `redis.windows-service.conf` 파일을 `redis.conf` 로 이름 변경
* 관리자 권한으로 cmd 실행 후 레디스 설치 위치에서
  * `> redis-server --service-install redis.conf --service-name redis6379`
  * 위의 명령 실행 시, 윈도우의 서비스에 redis6379가 등록됨
* 레디스 서버 시작
  * `> redis-server --service-start --service-name redis6379`

* 레디스 cli 시작하고 사용하기
  * `> redis-cli -p 6379`
  * `> set key value`
    * OK
  * `> get key`
    * "value"
  * `> exit`

* 레디스 서버 종료
  * `> redis-server --service-stop --service-stop --service-name redis6379`
* 레디스 서비스 해제
  * `> redis-server --service-uninstall --service-name redis6379`