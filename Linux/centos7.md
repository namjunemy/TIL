# CentOS 7

### 배포판 버전 확인

```shell
# grep . /etc/*-release
```

```shell
# rpm -qa *-release
```

###   

### 시간 설정 및 Time Zone 변경

* 확인

```shell
# date
```

* 서울 Time Zone 정보 위치

```shell
# /usr/share/zoneinfo/Asia/Seoul
```

* 서버 시간 설정파일에 심볼릭 링크 추가

```shell
# mv /etc/localtime /etc/localtime_backup
# ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
```

* timedatectl 사용

```shell
# timedatectl set-timezone Asia/Seoul
```

###   

### 서비스 관리

CentOS 7 부터 서비스 데몬 관리 방법이 달라졌다. 기존의 init system에서 systemd로 기본 시스템 관리 데몬이 변경되었기 때문이다.

> 서비스 예시 : elasticsearch

#### 서비스 등록

```shell
# systemctl mask elasticsearch.service
```

#### 서비스 삭제

```shell
# systemctl unmask elasticsearch.service
```

#### 서비스 상태 확인

```shell
# systemctl status elasticsearch.service
```

#### 서비스 구동

```shell
# systemctl start elasticsearch.service
```

#### 서비스 중지

```shell
# systemctl stop elasticsearch.service
```

#### 서비스 재실행

```shell
# systemctl restart elasticsearch.service
```

#### 부팅시 서비스 자동 시작

```shell
# systemctl enable elasticsearch.service
```

#### 부팅시 서비스 자동 시작 비활성화

```shell
# systemctl disable elasticsearch.service
```

#### 서비스 목록

* 설치된 모든 unit파일

```shell
# systemctl list-unit-files
또는
# systemctl list-units
```

#### 조건에 맞는 서비스

* 구동에 실패한 서비스

```shell
# systemctl list-units --state=failed
```

* 모든 active 서비스 목록

```shell
# systemctl list-units --state=active
```

* 서비스중에 상대가 running인 목록

```shell
# systemctl list-units --type=service --state=running 
```

#### 특정서비스 상태조회

* active 상태 조회

```shell
# systemctl is-active elasticsearch
```

* enable 상태 조회(부팅시 실행되는 서비스인지)

```shell
# systemctl is-enabled elasticsearch
```



