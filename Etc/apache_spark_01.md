# Apache Spark

>KITRI - 빅데이터와 스파크 프레임워크
>
>한빛미디어 - 아파치 스파크 입문

## 환경 설치

* VirtualBox Ubuntu 16.04
* 메모리 12GB / CPU 2 or 4 (메모리는 최소 8이상 권장) 
* 하드 디스크(기존 or 새로 생성)
* hadoop, hive, spark 설치
* Ambari - centos에서 최신버전 제공, 환경설정등 편리

### hadoop

* 하둡 버전이 올라갈수록 hadoop + hive는 데이터 저장 용도로 쓰고, spark등 다른 tool들을 이용

* [hadoop installed PATH]/etc/hadoop
  * 환경 설정 파일들 위치
* vi core-site.xml
  * hdfs value => hdfs://localhost:54310
* vi hdfs-site.xml
  * replication 설정

### hive

* hive를 통해서 hadoop을 sql로  쉽게 접근할 수 있음

* [hive installed PATH]/conf
  * 환경 설정
* vi hive-site.xml
  * javax.jdo 설정을 통해서 mysql과 연동

### spark

* spark 1.6 버전과 2.x 버전은 호환성이 떨어진다.
* [spark installed PATH]/sbin
  * .sh 파일들 위치
* [spark installed PATH]/conf

### shell script

* start-all.sh를 생성하여 필요한 환경을 전체 start함
  * namenode
  * datanode
  * secondarynamenode
  * yarn daemons
  * resourcemanager
  * nodemanager
* stop-all.sh를 생성하여 전체 stop

### 관리 콘솔 접근

* http://localhost:50070
  * 관리
* http://logalhost:54310

## 실행

### hdfs

* 하둡 파일 시스템에 저장된 디렉토리 구조 확인
  * hdfs dfs -ls /test
* 저장된 데이터 내용 보기
  * hdfs dfs -cat /test/names/names.csv
* 하둡 파일 시스템에 디렉토리 만들기
  * hdfs dfs -mkdir /test/kitrispark
  * 권한 설정
  * hdfs dfs -chown [대상 디렉토리]
* aa.txt 파일 hdfs에 넣기
  * hdfs dfs -put /home/hadoop_user/kitrispark/aa.txt /test/kitrispark

### hive

* concole에서 `$ hive` 명령을 통해 접속
* 위에서 한 설정을 바탕으로 hdfs데이터를 hive를 통해 MySQL DB처럼 사용 가능
  * 이것을 그대로 흉내낸 것이 Spark SQL
* `hive> select count(1) from names;`
* `hive> quit;`

### spark

빅데이터 처리를 위한 오픈소스 병렬분산처리 플랫폼. 스파크는 복수의 컴포넌트로 구성된다. 병렬분산처리 엔진에 해당하는 스파크 코어를 중심으로, SQL 처리용 라이브러리(스파크 SQL), 스트림처리용 라이브러리(스파크 스트리밍), 머신러닝용 라이브러리(MLlib), 그래프처리용 라이브러리(그래프X)

* `$ /usr/local/spark/bin/spark-shell`
* spark version
  * 2.2.1
* scala version
  * 2.11.8
* 스파크 코어
  * 스파크 코어는 데이터 소스로 HDFS 뿐만아니라 hive, HBase, PostgreSQL, MySQL, CSV 파일 등도 받아들일 수 있다.

### 사용법

### scala

* https://www.tutorialspoint.com/scala/scala_quick_guide.htm

```scala
val s = "hello scala"
print(s)

...
```

### hive

* https://www.tutorialspoint.com/hive/hive_quick_guide.htm
* CREATE TABLE

```sql
hive> CREATE TABLE IF NOT EXISTS employee ( eid int, name String, salary String, destination String)
> COMMENT ‘Employee details’
> ROW FORMAT DELIMITED
> FIELDS TERMINATED BY ‘\t’
> LINES TERMINATED BY ‘\n’
> STORED AS TEXTFILE;
```

## Spark 처리 모델

### RDD(Resilient Distributed Datasets)

* 스파크의 데이터처리는 RDD라는 자료구조를 이용한다.
* RDD는 대량의 데이터를 요소로 가지는 분산 컬렉션이다. 거대한 배열과 리스트 등의 자료구조를 생각하면 이해하기 쉽다
* RDD는 여러 머신으로 구성된 클러스터 환경에서의 분산처리를 전제로 설계되었고, 내부는 파티션이라는 단위로 나뉜다.
* 스파크에서는 이 파티션이 분산 처리 단위다.
* RDD를 파티션 단위로 여러 머신에서 처리하므로 한 대의 머신으로 처리할 수 있는 것보다 더 큰 데이터를 다룰 수 있다.
* 예를 들어, HDFS 등의 분산 파일 시스템에 있는 파일 내용을 RDD로 로드하고, RDD를 가공하는 형식으로 대량의 데이터를 분산처리 할 수 있다.
* 스파크에서는 이런 가공을 **Transformation** 이라고 한다.
* 그리고 RDD의 내용에 따라 **Action**이라는 처리를 적용하여 원하는 결과를 얻는다.
* RDD = Transformation + Action
* RDD는 immutable하며, 생성이나 변환이 지연 평가되는 성질이 있다.

### Transformation(변환)

* filter
  * 요소를 필터링한다.
* map
  * 각 요소에 동일한 처리를 적용
* flatmap
  * 각 요소에 동일한 처리를 적용하고 여러 개의 요소를 생성한다.
* zip
  * 파티션에 있는 요소의 수도 같은 두개의 RDD를 조합해 한쪽의 요소 값을 키로, 다른 한쪽의 요소 값을 밸류로 가지는 pair로 만든다.
* reduceByKey
  * 같은 키를 가지는 요소를 집약처리(Aggregation)한다.
* join
  * 두 개의 RDD에서 같은 키를 가지는 요소끼리 조인한다.

### 데이터 처리

### hive

* hdfs 디렉토리 구조 확인

  * `$ hdfs dfs -ls /test`

* hdfs 디렉토리 생성

  * `$ hdfs dfs -mkdir /test/kitrinames`

  * `$ hdfs dfs -put [PATH]/names.csv /test/kitrinames`

* hdfs에 저장된 csv 파일 확인

  * `$ hdfs dfs -cat /test/kitrinames/names.csv`

* 저장된 파일 삭제

  * `$ hdfs dfs -rm /test/kitrinames/*`

* 저장된 hdfs의 데이터들을 MySQL query를 통해서 다룰 수 있다.

## Spark 설치

> ubuntu or centos 등 각자 환경에 맞게 설치

### HDFS

* 하둡 분산 파일 시스템으로, 클러스터 환경에서의 작동을 전제로 설계되었다. NameNode라고 하는 마스터 노드와 DataNode라고 하는 복수의 워커 노드로 구성된다.
* NameNode
  * 마스터 노드, HDFS상에 보존되는 파일의 메타데이터와 보존된 파일의 분할된 조각이 어떤 DataNode에서 관리되는지 등의 정보를 관리한다.
* DataNode
  * HDFS상에 보존된 파일의 블록을 관리한다.

### YARN

* 하둡의 클러스터 관리 시스템으로, 스파크를 비롯한 각종 분산처리 프레임워크로 개발한 애플리케이션이 작동하는 환경을 제공.관리하는 기반이다. YARN은 Resource Manager라고 불리는 마스터 노드와 NodeManager라고 하는 여러 개의 워커 노드로 구성 된다.

## Spark 애플리케이션 개발과 실행

### sbt

스파크 애플리케이션은 소스코드를 컴파일하고 JAR 파일로 패키징해야 한다. 이 책에서는 sbt를 이용해 컴파일과 패키징을 실행한다. sbt는 스칼라와 자바로 기술된 소스코드를 컴파일하고, 라이브러리 의존 관계를 관리하며 패키지를 작성하는 등, 애플리케이션 개발 프로젝트에서 빌드 프로세스를 통합 관리하기 위한 툴이다.

* 프로젝트 생성
  * `$ sbt new sbt/scala-seed.g8`
* sbt 프로젝트의 디렉토리 구조
  * /
    * src
      * main
        * scala	
      * test
        * scala
    * lib
    * project
* 프로젝트 디렉토리 세팅 후 어셈블리 JAR 파일을 만들기 위한 명령
  * `$ sbt assembly`
* spark shell 실행
  * `$ /usr/local/spark/bin/spark-shell`

### hdfs에서 파일 읽어서 filtering 후 spark(scala)로 출력하기

* simple-words.txt

```shell
cat
dog
.org
cat
cat
&&
tiger
dog
100
tiger
cat
```

* `$ val af = "hdfs://localhost:54310/test/kitrinames/simple-words.txt"`
* `$ val tdd = sc.textFile(af)`
* `$ val wordRDD = tdd.filter(_.matches("""\p{Alnum} + """))`
* `$ val wordArray = wordRDD.collect`
* `$ wordArray.foreach(println)`

```shell
cat
dog
cat
cat
tiger
dog
100
tiger
cat
```