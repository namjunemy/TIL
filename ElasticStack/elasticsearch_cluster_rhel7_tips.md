# Elasticsearch Cluster Tips

> rhel 7.5(서비스 데몬 => systemd) 기반으로 Elasticsearch Cluster를 구성하면서 마주한 이슈 해결 방법

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

* `/usr/lib/systemd/elasticsearch.service` 파일에 아래의 내용 추가

```
LimitMEMLOCK=infinity
```

* 추가 후 daemon-reload

```shell
# systemctl daemon-reload
# systemctl restart elasticsearch
```

