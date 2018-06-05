# Elasticsearch Cluster Tips

> rhel 7.5(서비스 데몬 => systemd) 기반으로 Elasticsearch Cluster를 구성하면서 마주한 이슈 해결 방법과 사소한 팁

## hostname 변경

```shell
# hostnamectl set-hostname "custom-name"
```

## bootstrap check issue

* `/etc/elasticsearch/elasticsearch.yml 파일에 bootstrap.memory_lock: true 설정시 아래의 에러 발생`

```shell
[2018-06-04T14:17:27,702][ERROR][o.e.b.Bootstrap          ] [es-master] node validation exception
[1] bootstrap checks failed
[1]: memory locking requested for elasticsearch process but memory is not locked
```

* `/usr/lib/systemd/system/elasticsearch.service` 파일에 아래의 내용 추가

```
LimitMEMLOCK=infinity
```

* 추가 후 daemon-reload

```shell
# systemctl daemon-reload
# systemctl restart elasticsearch
```

## cluster에 포함된 노드 확인

```shell
# curl http://[server ip]:9200/_nodes/process?pretty
```

## cluster x-pack password 설정

* master 노드에 x-pack을 설치하고 setup-passwords를 수행할 때, discovery.zen.ping.unicast.hosts 설정을 하지 않고 default상태로 setup-passwords를 수행한다.(그렇지 않을 경우 404 error 발생)
* 패스워드 설정이 끝나면, discovery설정을 활성화 시킨다.
* data 노드에 x-pack을 설치하고 elasticsearch.yml에 discovery 설정이 마스터 노드로 되어있는 상태에서 elasticsearch 를 재실행 하고, cluster에 포함된 노드를 확인해서 성공적으로 클러스터 구성이 되었다면, 해당 data 노드는 클러스터의 password 설정을 따른다. 마찬가지로 나머지 data 노드들을 설정 한다.

## NFS 구성 issue

* 마스터 노드에서 여러개의 데이터 노드를 관리하기 위해서 NFS 구성 필수

* 이전 버전의 OS와 nfs설정이 살짝 달라졌다.
  * nfs-client의 /etc/fstab 설정시 여러 설정값들을 제외하고 sync를 사용하면 된다.
* 참조 링크 :
  * https://www.thegeekdiary.com/centos-rhel-7-configuring-an-nfs-server-and-nfs-client/
  * http://yangnoon.tistory.com/38

