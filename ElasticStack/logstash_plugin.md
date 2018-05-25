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
    columns =["host","group","service","db","web","was","ad","etc","platform"]
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

