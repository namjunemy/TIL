# Elasticsearch Getting Started

> 김종민 - Elastic 테크 에반젤리스트
>
> **Contents**
>
> 1. Elastic Stack 소개
> 2. Elasticsearch 상세
> 3. Demo
> 4. Reference

### Elastic Stack

* ELK로 많이 알려져 있지만, 2015년 Beats가 합류하고, Elastic사에서 Stack들을 계속 추가하고 있기 때문에 공식적으로 Elastic Stack으로 명명하고있다.
* Elasticsearch, Kibana, Logstash, Beats가 포함되며
* 100% 오픈소스이며, Apache 2 라이센스로 제공

### X - Pack

* Elastic Stack과 별개의 확장팩으로 배포하는 상용 플러그인
* 기능은 Security, Alerting, Monitoring, Reporting, Graph Analytics, Machine Learning

### Elastic Cloud

* Elaticsearch 클러스터를 Cloud에서 SaaS 형태로 몇번의 클릭만으로 만들 수 있다. 


### Elastic Cloud Enterprise

* 추가적으로 기업 또는 거대한 조직에서 필요한 클라우드 서비스가 있는 경우 Elastic Cloud Enterprise 서비스를 이용할 수 있다.
* IDC 또는 Private Network환경에서 Elastic Cloud Enterprise를 설치하면, 조직내의 서로 다른 부서에서 각각 다른 Elasticsearch환경이 필요하다고 할 때, 해당 환경에 맞는 Elastic 클러스터를 X-Pack이 포함된 상태로 생성할 수 있다.

![](https://github.com/namjunemy/TIL/blob/master/ElasticStack/img/elastic_cloud_enterprise_01.PNG?raw=true)

## Elasticsearch

Elasticsearch는 Elastic Stack의 심장이라고 불릴만큼 중요한 역할을 담당한다. Elasticsearch에 모든 데이터가 저장이 되며 검색, 쿼리 등 필요한 것들은 Elasticsearch에서 처리 된다.

### 특징

* Distributed, Scalable
* High-availability
* Multi-tenancy
* Developer Friendly
* Real-time, Full-text Search
* Aggregations

### Today's Developer Requirements

- Horizontal Scale
- Real-Time Data Availability
- Flexible Data Model
- Rapid Query Execution
- Sophisticated Query Langeage
- Schemaless

### Apache Lucene

* 하둡을 개발한 Doug Cutting에 의해 Java로 개발되었다.
* Elasticsearch는 기본적으로 Apache Lucene이라는 라이브러리를 사용한다.
* Lucene은 자체적으로 검색엔진을 만들 수 있지만, 라이브러리이기 때문에 이것을 Product로 개발하는 과정이 필요하다.
* 그렇게 만들어진 대표적인 Product가 **Apache Solr**와 **Elasticsearch**이다.

### Elasticsearch 클러스터링

* 데이터들이 대용량으로 처리가 되기 때문에 클러스터링 환경을 필요로 한다.
* Elasticsearch는 기본적으로 데이터를 샤드(Shard) 단위로 분리해서 저장한다.
* 샤드(Shard)는 Lucene 레벨의 검색 스레드이고, Elasticsearch의 실행 프로세스를 노드(Node)라고 표현한다.
* **Elasticsearch의 Shard와 Node**

![](https://github.com/namjunemy/TIL/blob/master/ElasticStack/img/elasticsearch_shard_node_01.PNG?raw=true)

* 노드를 여러개 실행시키면 이 노드들은 같은 클러스터로 묶여서 같은 시스템으로 동작을 한다.

![](https://github.com/namjunemy/TIL/blob/master/ElasticStack/img/elasticsearch_shard_node_02.PNG?raw=true)

* 샤드들은 각각의 노드들에 분배되어 저장된다.

![](https://github.com/namjunemy/TIL/blob/master/ElasticStack/img/elasticsearch_shard_node_03.PNG?raw=true)

* 무결성과 가용성을 위해 샤드의 복제본을 만든다. 같은 내용의 복제본과 샤드는 서로 다른 노드에 저장된다.

![](https://github.com/namjunemy/TIL/blob/master/ElasticStack/img/elasticsearch_shard_node_04.PNG?raw=true)

* 시스템 다운이나 네트워크 단절 등으로 유실된 노드가 생기면,

![](https://github.com/namjunemy/TIL/blob/master/ElasticStack/img/elasticsearch_shard_node_05.PNG?raw=true)

* 복제본이 없는 샤드들은 다른 살아있는 노드로 복제를 시작한다.

![](https://github.com/namjunemy/TIL/blob/master/ElasticStack/img/elasticsearch_shard_node_06.PNG?raw=true)

* 노드의 수가 줄어들어도 샤드의 수는 변함 없이 무결성을 유지한다.

![](https://github.com/namjunemy/TIL/blob/master/ElasticStack/img/elasticsearch_shard_node_07.PNG?raw=true)



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