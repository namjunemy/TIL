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

* text file -> hdfs -> hive create -> hive save의 흐름
* hive 1 -> hive 2 + spark sql 추세

## Spark

* scala 문법 사용 간결한 코딩
* lambda expression 등
* sbt new sbt/scala
* sbt compile
* sbt run
* sbt package
* 한번에 -> sbt assembly

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

* `$ val wordAndOnePairRDD = wordRDD.map(word => (word, 1))`
* `$ val wordAndOnePairArray = wordAndOnePairRDD.collect`

### RDD 요소를 키 단위로 집약처리하기

* `$ val wordAndCountRDD = wordAndOnePairRDD.reduceByKey((result, elem) => result + elem)`
* `$ val wordAndCountArray = wordAndCountRDD.collect`

## 구조화된 데이터셋 처리하기: Spark SQL

* **DataFrame** - untyped DataSet
  * DataFrame으로 만들었을때의 장점은 SQL 쿼리로 편리하게 데이터 처리를 할 수 있다.
* **DataSet** - typed -> table
* `$ rdd -> rdd.toDF("a", "b")`
* df.rdd -> rdd로 변환
* case class  - java의 DTO, VO 역할
* df.printSchema