# Logstash plugin

* [Logstash Reference](https://www.elastic.co/guide/en/logstash/current/index.html)
  * Input plugins / [doc](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)
  * Output plugins / [doc](https://www.elastic.co/guide/en/logstash/current/output-plugins.html)
  * Filter plugins / [doc](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html)

## csv file import

```shell
input {
  file {
    path => "/root/elastic/vm_list/201805/20180502_vm_all.csv"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}
filter {
  csv {
    separator => ","
    columns => ["host","group","service","db","web","was","ad","etc","platform"]
  }
}
output {
  elasticsearch {
    hosts => ['localhost:9200']
    user => "elastic"
    password => "password"
    index => "201805_all_vm_list"
  }
  stdout { codec => rubydebug }
}
```

## tcp input

* `$ echo "hello" | nc localhost 9999` 명령을 통해 tcp input 확인

```shell
input {
  tcp {
    port => 9999
  }
}
output {
  elasticsearch {
    hosts => ['localhost:9200']
    user => "elastic"
    password => "password"
    index => "tcp"
  }
  stdout { codec => rubydebug }
}
```

## logstash 실행

* 작성한 logstash conf 파일을 로드, 로그스태시 실행

```text
Could not find logstash.yml which is typically located in $LS_HOME/config or /etc/logstash. ~~~~~~
```

  * 위의 에러 로그가 발생하면 `--path.settings=/etc/logstash` 실행 옵션 추가

```shell
$ /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/[파일명].conf 
```

