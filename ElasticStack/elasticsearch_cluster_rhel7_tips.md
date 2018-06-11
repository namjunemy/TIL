# Elasticsearch Cluster Tips

> rhel 7.5(서비스 데몬 => systemd) 기반으로 Elasticsearch Cluster를 구성하면서 마주한 이슈 해결 방법과 사소한 팁

### Red Hat Subscription-Manager

* `$ subscription-manage register --force --auto-attach`
* `$ yum repolist all`

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

## user add(use cli)

* x-pack이 install 되어 있는 상태에서

* `/usr/share/elasticsearch/bin/x-pack/users useradd [아이디] -p [패스워드] -r superuser`
* superuser에는 생성을 원하는 권한

### Metricbeat

> https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-getting-started.html

* 설정시 주의 사항
  * 마스터 노드와 데이터 노드가 x-pack을 통해 TLS/SSL 설정이 되어있을 때, metricbeat 에서 마스터 노드에 설치된 kibana의 end-point를 설정할 경우 내부 kibana의 host를 내부 ip로 변경해야 한다. x-pack에 등록된 ip가 외부에 공개되지 않은 10.x.x.x 대역의 ip이기 때문이다.
  * 따라서, 외부 접근이 허용된 kibana의 host를 내부 ip로 수정하고 metricbeat 의 kibana end-point 역시 내부 ip로 설정한 뒤에 metricbeat 의 대시보드를 set-up 한다. 
* metricbeat.yml kibana end-point 설정

```yml
...

setup.kibana.host: "10.5.25.5:5601"
setup.kibana.protocol: "https"
setup.kibana.ssl.enabled: true
setup.kibana.ssl.certificate_authorities: ["/usr/share/elasticsearch/cert/ca/ca.crt"]
setup.kibana.ssl.certificate: /usr/share/elasticsearch/cert/l-cloud-es/l-cloud-es.crt
setup.kibana.ssl.key: "/usr/share/elasticsearch/cert/l-cloud-es/l-cloud-es.key"

...
```

* metricbeat 대시보드 set-up
  * `$ metricbeat setup --dashboards`