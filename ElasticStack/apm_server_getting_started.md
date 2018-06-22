# Elastic Stack APM Server 

> https://www.elastic.co/guide/en/apm/server/6.2/index.html
>
> v6.2.4, rhel7.5 기준

## Install

* download and install public signing key

```bash
$ sudo rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
```

* create file `.repo` (for example, `elastic.repo`) extension in your `/etc/yum.repos.d` 

```shell
[elastic-6.x]
name=Elastic repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

* install

```shell
$ yum install apm-server-6.2.4
```

* configure to start automatically during boot, run

```shell
$ systemctl daemon-reload
$ systemctl enable apm-server
```

## Config

* `/etc/apm-server/apm-server.yml`

```yml
apm-server:
  host: "[Coordinator Node IP]:8200"

...

output.elasticsearch:
  hosts: ["[Coordinator Node IP]:9200"]
  username: "elastic"
  password: "ldcc!2626"
```

## Run

```shell
$ systemctl start apm-server
```



