# Elasticsearch 시작하기

> 김종민 - Elastic 에반젤리스트

### Contents

* Elastic 소개
* Elasticsearch 데모

## 특징

* Horizontal Scale
* Real-Time Data Availability
* Flexible Data Model
* Rapid Query Execution
* Sophisticated Query Langeage
* Schemaless

### Elastic Stack

* ELK로 많이 알려져 있지만, 공식적인 명칭은 Elastic Stack
* Elasticsearch, Kibana, Logstash, Beats가 포함되며
* 100% 오픈소스이며, Apache 2 라이센스로 제공

### X - Pack

* Elastic Stack과 별개로 배포하는 상용 플러그인
* 기능은 Security, Alerting, Monitoring, Reporting, Graph Analytics, Machine Learning

### Elstic Cloud

* Elaticsearch 클러스터를 Cloud에서 SaaS 형태로 몇번의 클릭만으로 만들 수 있다.


## Elasticsearch Demo

> Web : www.elastic.co
>
> Products : https://www.elastic.co/products



### 다운로드

* https://www.elastic.co/kr/downloads/elasticsearch 에서 OS에 맞는 파일을 다운로드 한다.
* 압축을 풀고 실행한다. 실행하기 전에 Java가 설치 되어 있는지 확인해야 한다.
* 압축이 풀린 폴더로 이동하면, 아래와 같은 파일구조로 구성되어있다.
  * bin
    * 실행가능한 binary파일들이 위치한다. 
    * linux/unix 환경에서는 elasticsearch 파일을
    * windows 환경에서는 elasticsearch.bat 파일 또는 .exe을 실행하면 된다.
  * config
    * elasticsearch.yml
      * elasticsearch의 설정에 관련된 파일이다.
      * cluster명, node명, 데이터가 저장될 Path, Network host설정
        * Network host를 설정하지 않으면, default는 localhost로 설정된다.
        * elasticsearch의 노드가 localhost로 실행이 되면, 개발자 모드로 실행되서 시작단계에서 bootstrap체크 등을 하지 않는다.
        * 반면에, network host를 설정하게 되면 실제로 clustering을 체크하게 되기 때문에, 시작단계에서 다양한 체크가 들어가게 된다.
      * elasticsearch는 기본적으로 9200번 포트를 사용하여 client와 통신하며, 변경 가능하다.
      * clustering을 위해서 discovery나 unicast설정도 하게 된다. 더 자세한 내용은 documentation에 설명되어있다.
    * jvm.options
      * elasticsearch가 실행할 수 있는 자바 힙 메모리를 설정한다.
      * Xms와 Xmx를 동일하게 설정 해야 메모리 스왑이 적게 일어난다.
    * log4j2.properties
      * elasticsearch에서는 로그를 log4j로 남기기 때문에, 로그를 수정하려면 이 파일을 수정하면 된다.
  * lib
  * modules
  * plugins

## Elasticsearch 실행

* `$ elasticsearch설치위치/bin/elasticsearch` 명령을 통해 실행(Linux)
* 로그에는 elsticsearch.yaml 파일에서 설정한 JVM arguments가 표시되고, 통신을 위해 사용되는 포트들이 표시된다. 기본적으로 9300번 포트를 통해 노드들끼리의 통신이 이루어지고, 9200번 포트를 통해서 client와 통신을 하게 된다.
* elasticsearch의 정상적인 실행 여부를, 다음과 같은 명령어를 통해 확인할 수 있다.
  * `$ curl localhost:9200` 명령을 전송하면, elasticsearch의 현재 버전과 클러스터 정보, 노드의 이름 등이 JSON형식으로 리턴 된다. 