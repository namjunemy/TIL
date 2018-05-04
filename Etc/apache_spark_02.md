# Apache Spark 02

> KITRI - 빅데이터와 스파크 프레임워크
>
> 한빛미디어 - 아파치 스파크 입문

## hdfs

- hdfs 디렉토리 구조 확인
  - `$ hdfs dfs -ls /test`
- hdfs 디렉토리 생성
  - `$ hdfs dfs -mkdir /test/kitrinames`
  - `$ hdfs dfs -put [PATH]/names.csv /test/kitrinames`
- hdfs에 저장된 csv 파일 확인
  - `$ hdfs dfs -cat /test/kitrinames/names.csv`
- 저장된 파일 삭제
  - `$ hdfs dfs -rm /test/kitrinames/*`

## hive

- text file -> hdfs -> hive create -> hive save의 흐름
- hive 1 -> hive 2 + spark sql 추세

## Spark

- scala 문법 사용 간결한 코딩
- lambda expression 등
- sbt new sbt/scala
- sbt compile
- sbt run
- sbt package
- 한번에 -> sbt assembly

## 기본 API를 이용한 프로그래밍

### 텍스트 파일로부터 RDD 생성

- `$ val af = "hdfs://localhost:54310/test/kitrinames/simple-words.txt"`
- `$ val tdd = sc.textFile(af)`

### RDD 요소 필터링

- `$ val wordRDD = tdd.filter(_.matches("""\p{Alnum} + """))`
- `$ val wordArray = wordRDD.collect`

### 요소 출력

- `$ wordArray.foreach(println)`

### RDD 요소 가공

- `$ val wordAndOnePairRDD = wordRDD.map(word => (word, 1))`
- `$ val wordAndOnePairArray = wordAndOnePairRDD.collect`

### RDD 요소를 키 단위로 집약처리하기

- `$ val wordAndCountRDD = wordAndOnePairRDD.reduceByKey((result, elem) => result + elem)`
- `$ val wordAndCountArray = wordAndCountRDD.collect`

## 구조화된 데이터셋 처리하기: Spark SQL

- **DataFrame** - untyped DataSet
  - DataFrame으로 만들었을때의 장점은 SQL 쿼리로 편리하게 데이터 처리를 할 수 있다.
- **DataSet** - typed -> table
- `$ rdd -> rdd.toDF("a", "b")`
- df.rdd -> rdd로 변환
- case class  - java의 DTO, VO 역할
- df.printSchema

## Spark Stream - 연속적인 데이터의 흐름

- 스트림 데이터의 입력
  - 스파크 스트리밍으로 스트림 처리를 구현할 때는 먼저 데이터소스로부터 데이터를 읽어 들이는 정의를 하고, 그 데이터를 입력으로 하는 DStream을 정의한다.
  - 데이터소스의 대표적인 예
    - 파일로부터 데이터를 취득
    - 네트워크 소켓으로부터 UTF-8의 바이트 열을 읽어 들이는 기능
    - Akka의 액터(actor)를 이용해 데이터를 취득하는 기능
  - 고급 데이터소스를 이용하면 메시징 시스템이나 데이터 로더로부터 데이터를 취득할 수 있다. 스파크의 external 패키지에는 다음과 같은 각종 데이터소스를 위한 기능들이 포함되어 있다.
    - Kafka
    - Flume
    - Kinesis
    - Twitter
    - mqtt
    - zeromq
- DStream 변환
  - 데이터소스에서 데이터를 읽어 들이기 위한 DStream을 정의하면, 그 변환처리를 정의할 수 있게 된다.
  - DStream은 RDD의 변환 API와 비슷한  API를 이용할 수 있다.
    - map
    - flatMap
    - filter
- DStream 출력
  - print()
    - 10건만 선택해서 드라이버의 표준 출력에 결과를 표시
  - saveAsTextFiles(prefix, [suffix])
    - DStream의 콘텐츠를 복수 개의 텍스트 파일에 보존

## 머신러닝 MLlib

### MLlib이 제공하는 머신러닝 알고리즘

- 분류와 회귀
- 협업 필터링
- 클러스터링
- 차원 축소
- 특징 추출/변환
- 빈발 패턴 마이닝