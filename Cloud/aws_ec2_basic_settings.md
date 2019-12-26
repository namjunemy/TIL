# EC2 서버 생성, 접속시 필수 설정

> 아마존 리눅스 1 서버 기준
>
> * EC2 ssh 접속 로직 간소화
>
> * Java8 설치
> * Timezone 변경
> * 호스트네임 변경

## EC2 서버 접속 설정

> Mac 기준

* EC2 서버로 SSH 접속하려면 매번 pem파일과 ip를 입력해야 한다.

    * `$ ssh i {pem 키 위치} {public Ip주소}`

* ~/.ssh/ 디렉토리로 pem 파일을 옮겨 놓으면 ssh 실행 시 pem 키 파일을 자동으로 읽어 접속할 수 있다.

    * `$ cp {pem키 위치} ~/.ssh/`

* pem키 권한 변경

    * `$ chmod 600 ~/ssh/{pem 키}`

* pem키가 있는 ~/.ssh 디렉토리에 config 파일 생성

    * `$ vim ~/.ssh/config`

        ```shell
        # myServiceName용 접근 설정
        Host myServiceName
           HostName {public Ip 주소}
           User ec2-user
           IdentityFile ~/.ssh/myServiceName.pem
        ```

* config 파일 권한 변경

    * `$ chmod 700 ~/.ssh/config`

* 아래와 같이 간소화된 명령어로 EC2 접근

    * `$ ssh myServiceName`

## Java8 설치

* 아마존 리눅스 1의 경우 기본 자바 버전이 7, 프로젝트 버전에 맞게 변경.

* `$ sudo yum install -y java-1.8.0-openjdk-devel.x86_64`
* `$ sudo /usr/sbin/alternatives --config java`
* Java8 선택 후 변경 완료 확인: `$ java -version`
* Java7 제거: `$ sudo yum remove java-1.7.0-openjdk`

## Timezone 변경

* EC2 서버의 기본 타임존은 UTC다. 한국과는 9시간 차이 발생. 애플리케이션에서 생성되는 시간도 모두 9시간 차이나게 됨. 필수 변경 사항.
* `$ sudo rm /etc/localtime`
* `$ sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime`
* 변경 확인 `$ date`

## Hostname 변경

* 여러 서버를 관리 중일 경우 IP만으로 어떤 서비스의 서버인지 확인이 어려움

* 해당 서비스를 표현하기 위해 HOSTNAME 변경

* `$ sudo vim /etc/sysconfig/network`

* 변경 전

    ```shell
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    NOZEROCONF=yes
    ```

* 변경 후

    ```shell
    NETWORKING=yes
    HOSTNAME=myServiceName
    NOZEROCONF=yes
    ```

* `$ sudo reboot`

* hostname 변경 확인

* **추가 설정 사항**
    * 호스트 주소를 찾을 때 가장 먼저 보는 /etc/hosts에 변경한 hostname을 등록해야 한다.
    * 이유는 hostname이 /etc/hosts에 등록되지 않아서 장애가 발생할 수 있기 때문이다.
        * [우아한형제들 기술 블로그 - 빌링 시스템 장애, 이러지 말란 Maria~](http://woowabros.github.io/experience/2017/01/20/billing-event.html)
    * `$ sudo vim /etc/hosts`
    * hostname 추가 등록
        * `127.0.0.1 myServiceName`
    * curl로 확인
        * `$ curl myServiceName`
            * 정상 등록 됐다면 80 포트 접근 불가 메시지
                * `curl: (7) Fail to connect .... port 80: Connection refused`
            * 비정상이라면 호스트 찾지 못했음을 나타내는 메시지
                * `Could not resolve host: ...`

## Reference

* `스프링 부트와 AWS로 혼자 구현하는 웹 서비스 - 이동욱`
* http://woowabros.github.io/experience/2017/01/20/billing-event.html