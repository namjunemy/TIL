# Kafka 2019 Conference 후기

> 2019-10-18
>
> 줌인터넷 DI팀의 카프카 컨퍼런스 후기 요약

## 고승범 : How to utilize KAFKA more efficiently

- 주제 : Kafka를 더 효율적으로 사용하기 위해선 default로 설정된 옵션을 어떻게 바꿔야 하는지

- Broker Options

  1. log.retention.hours

     - Broker는 디스크에 데이터를 일정 시간동안 보관하며, 최신 버전 기준 2.3(현재 세션 설명 기준) kafka default 값으로는 일주일(168시간)로 설정되어 있습니다.
     - 초당 10,000개의 1KB 메시지가 7일동안 3개의 복제 수준을 가진 토픽들에 전송되었을 경우, 168TB의 디스크 공간이 필요합니다. 사용 목적에 따라 조금은 다르겠지만 대부분 브로커 메시지를 실시간으로 가져오기 때문에 일주일의 보관 기간은 비효율적이며 주말 기간을 고려하여 최대 3일(72시간)을 권장하고 있습니다.

  2. delete.topic.enable

     ![img](https://github.com/namjunemy/TIL/blob/master/SeminarAndConference/img/kafka_seoul_2019/kafka_01.png?raw=true)

     - 카프카 topic 삭제 가능 여부 옵션으로, false로 설정할 시, 특정 토픽(perter-topic)을 삭제하려고 했을 때, 실제로 삭제되지는 않고 삭제 표시(*Topic peter-topic is marked for deletion*)만 됩니다.
     - true로 설정하기를 권장하고 있으며, 검색해본 결과 버전 1.0.0 부터 기본 값으로 true를 채택하고 있습니다
       - 참조 https://kafka.apache.org/documentation/#upgrade_100_notable

  3. allow.auto.create.topcis

     - producer가 kafka cluster에 존재하지 않은 토픽으로 메시지를 전송하였을 경우, auto.create에 관련된 다른 옵션들을 참고하여 자동으로 토픽을 생성해줄지 선택하는 옵션입니다.
     - 무분별한 토픽 생성과 관리가 어려워지므로 권장 옵션은 false입니다. (기본 값은 true입니다)

  4. log.dirs

     - 기본 값으로 /tmp/kafka-logs를 가지고 있습니다.
       - /tmp 경로는 임시 파일을 저장하는 용도로 사용되고 있으며, 많은 운영체제에서 부팅 과정 중에 해당 경로 하위 파일들을 삭제하거나 혹은 특정 프로그램이 주기적으로 파일들을 삭제하고 있습니다
     - 이것도 마찬가지로 사용 목적에 따라 다르겠지만, 카프카를 장난감 프로젝트로 가볍게 다룰 예정이면 생성한 카프카 데이터 삭제를 신경쓰지 않아도 되는 기본 값이 좋겠지만 실제로 카프카를 이용해 서비스를 운영할 경우에는 위와 같이 다른 제 3자에 의해 삭제되는 경로가 아닌 /data와 같은 경로를 권장하고 있습니다.

  5. broker.rack

     - 물리적으로 분리된 여러 Rack에브로커 서버들이 위치해있을 때, 각 브로커 서버가 자신이 어떠한 Rack에 위치해있는지를 설정 값으로 적용했을 경우, 파티션 복제 과정 중에 이러한 Rack 정보를 활용하여 더 나은 fault tolerance를 제공해줄 수 있습니다.
     - 기본 값은 null값을 가지고 있습니다.

  6. replication.factor

     - 토픽의 partition의 복제 계수를 나타내며, 기본 값으로는 1을 가집니다. 따라서, partition을 가지고 있는 서버가 fail되었을 때, 해당 partition에 저장된 데이터를 잃어버릴 수 있으므로 장애 대응을 위해3개의 복제 계수를 설정할 것을 권장하고 있습니다.

- 결론

  - default 값을 사용하지 말자

## 강한구 : Producer와 Consumer

- Producer
  - Producer Node는 **Accumulator**와 **Network thread**들로 구성되어 있고 각 Network thread들은 Broker와 연결되어 있음
    - **Accumulator**
      - send한 Record를 메모리(RecordBatch)에 **저장**하는 역할을 담당
    - **Network thread**
      - Accumulator에 쌓인 RecordBatch를 Broker로 **전송**하는 역할을 담당
  - Accumulator Config
    - batch.size (default : 16KB)
      - Record를 전송할 topic의 partition 별로 모아 batch로 작업하기 위한 사이즈
    - buffer.memory (default : 32MB)
      - Accumulator에서 사용하는 메모리 크기
    - default 설정이라면 최대 2048개의 record batch가 생성
    - Broker로 보내는 속도보다 쌓이는 속도가 더 빠르다면 buffer.memory만큼 채워지고 설정된 max.block.ms 시간만큼 블락
  - Network tread Config
    - linger.ms (default : 0)
      - Network thread에서 accumulator에 쌓인 데이터를 가져올 때 대기하는 시간
      - 약간의 대기를 준다면 record batch가 조금 더 찬 상태에서 전송할 수 있음
      - 0이 아닌 값을 설정하면 broker 전송 수는 줄지만 latency 또한 감소
    - max.request.size (default : 1MB)
      - Broker로 한번 전송할 때의 최대 크기
    - max.request.size/batch.size
      - 한번에 전송 가능한 최대 record batch 개수
      - 두 값이 default인 경우 64개
    - max.in.flight.requests.per.connection
      - ack를 받지 않고 전송할 수 있는 최대 request 개수
        - ack에 대한 설정은 acks 옵션으로 설정 가능
      - retries가 설정된 상태에서 max.in.flight.requests.per.connection 값이 2 이상이면 메시지의 순서가 바뀔 수 있다.
- Broker
  - Topic name - partition 의 폴더 구조로 데이터를 저장
  - Disk 헤더 개수, 장비 노후화를 고려하여 하나의 큰 용량보다는 여러 개의 작은 용량 disk 구성 권장
    - ex) 4TB 한개보다는 1TB 4개를
  - Segment 단위로 파일을 저장하며 *.index, *.log, *.timeindex로 구성
    - *.index
      - 해당 메시지의 offset과 log 파일에서의 위치 저장
      - 파티션에서 오프셋에 해당하는 위치를 빠르게 찾기 위해 사용
    - *.log
      - offset과 position, 크기 및 실제 데이터
    - *.timeindex
      - index와 유사하나 offset 대신 timestamp 값으로 인덱싱.
      - timestamp 기준으로 메시지를 찾아 읽거나, 메시지의 rolling, retention에 활용
    - Broker Config
      - index.interval.bytes
        - record.batch가 log파일에 쓰일 때마다 인덱싱 될 간격
- Consumer
  - Fetcher와 Coordinator로 구성
  - **Fetcher**는 poll 함수가 실행되면 적절한 크기의 record들을 리턴하고 내부에 record들이 없다면 broker에게 record들을 요청하고 저장하는 역할
  - **Coordinator**는 어떤 토픽, 파티션을 consume 할지 broker의 group coordinator와 통신하며 heartbeat, offset commit, consumer group join 등의 역할을 수행
  - Consumer Config
    - max.poll.records (default : 500)
      - fetcher로 부터 가져올 record의 최대 개수
    - max.poll.interval.ms (default : 5분)
      - poll함수를 실행 해야할 최대 시간 간격
      - 설정된 시간 안에 poll 함수를 실행하지 않으면 consumer에 문제가 있다고 판단, 해당 consumer를 consumer group에서 제거 후 consumer group은 rebalance 실행
    - max.partition.fetch.bytes (default : 1MB)
      - 하나의 topic + partition에서 가져올 데이터의 최대 크기
    - fetch.min.bytes (default : 1byte)
      - 하나의 broker에서 가져올 데이터의 최소 크기
    - fetch.max.wait.ms (default : 500ms)
      - fetch.min.bytes 만큼 쌓일때까지 기다리는 최대 시간
    - fetch.max.bytes (default : 50MB)
      - 하나의 broker 에서 가져올 데이터의 최대 크기record batch 단위로 가져오기 때문에 대략적인 값
  - Consumer Rebalance
    - 아래의 조건에서 일어나는 consumer의 partition 재분배 상황
      - Partition의 증가
      - Consumer node 삭제/추가
    - group.initial.rebalance.delay.ms 내에 각 consumer가 join 하지 못할 경우 해당 consumer group에서 제외
  - Consumer Group offset
    - Ver.0.9 미만에서는 zookeeper에 consumer offset을 저장
      - Consumer node가 많아지면 zookeeper의 부하 증가
    - Ver.0.9 이상에서는 __consumer_offset 토픽을 사용하여 offset 정보 record를 produce
      - compaction을 통하여 offset을 쉽게 찾을 수 있도록 함
  - Log Manager
    - log.cleaner.threads (default : 1)
      - Log compaction 작업을 할 thread의 수
    - log.cleanup.policy (default : delete)
      - delete와 compact 두 가지 모드가 존재
        - delete
          - 시간 설정이나 크기 설정에 따라 주기적으로 log segment를 삭제
        - compact
          - 압축을 통한 불필요한 레코드 삭제, 토픽 단위로 설정가능
    - log.retention.ms (default : 7일)
      - Log segment가 보관 될 시간, 토픽 단위로 설정 가능
    - log.retention.bytes (default : -1)
      - 삭제하기 전에 보관할 partition 당 log 수, log의 시간이나 크기 제한에 도달하면 삭제, 토픽 단위로 설정 가능
    - log.retention.check.interval.ms (default : 5분)
      - log 를 삭제할 대상을 확인하는 주기
    - log.cleaner.min.cleanable.ratio (default : 0.5)
      - log compactor가 log를 정리하려고 시도하는 빈도
    - log.segment.delete.delay.ms (default : 1분)
      - log segment를 삭제하기 전에 대기하는 시간
    - log.cleaner.min.compaction.lag.ms (default : 0)
      - 메시지가 log에서 압축되지 않은 상태로 유지되는 최소 시간

## 이동진 : Kafka Strems: Interactive Queries

Kafka Streams의 Interactive Query를 짧게 소개한 세션입니다.

발표 자료에 Interactive Query의 코드 설명이 잘 되어 있고 개인적으로 state store를 몰라서 내용이 이해가 안되는 부분이 있어 후기에서는 발표 내용보다는 Kafka Streams와 state store에 대해 조사하고 정리했습니다.

먼저 Kafka Streams를 살펴보겠습니다. Kafka Streams는 Kafka에 저장된 데이터를 처리하기 위한 클라이언트 라이브러리입니다. 데이터 처리에서 이미 사용하던 Kafka Consumer API와 다른 점은 프로그램 개발이 더 간단하다는 것입니다. Consumer API는 데이터 처리뿐만 아니라 받은 만큼 처리했다는 것을 알리는 커밋(commit), 장애 복구를 위한 기능 등 신경써야 할 것이 많습니다. 반면에 Kafka Streams는 join(), count(), map() 등의 DSL을 지원하고, 내부에 중간 처리 결과를 저장하는 key/value store가 있으며 key/value store의 변경내역을 자동으로 Kafka의 changelog topic에 저장하기 때문에 stateful 데이터 처리나 failover를 쉽게 개발할 수 있습니다. key/value store를 보통 state store라고 부릅니다.

Kafka Streams의 이해를 돕기 위해 Word count를 Consumer API와 Kafka Streams로 개발하는 상황을 예로 들겠습니다. 사용자가 "Hello World"를 입력하면 두 버전 모두 Hello = 1, World = 1 이 됩니다. 이어서 사용자가 "Hello Kafka"를 입력하면 Consumer API는 Hello = 1, Kafka = 1을 출력합니다. 그런데 Kafka Streams는 내부에 중간 처리 결과를 저장하고 있어 누적된 값인 Hello = 2, Kafka = 1을 출력합니다.

state store는 Kafka Streams 인스턴스의 로컬에 저장하기 때문에 만약 3개의 인스턴스가 있다면 3개의 state store가 있고 각각의 state store의 값이 모두 다릅니다. Interactive Query는 각각의 인스턴스가 가진 state store에서 저장하고 있는 중간 상태를 외부 API로 제공하거나 다른 호스트에서 실행 중인 인스턴스가 가진 값이 필요할 때 사용할 수 있습니다.

Interactive Query에서 요청 질의를 하거나 받는 코드는 아직 사용자가 직접 개발을 해야 합니다. [Lightbend에서 HTTP로 요청할 수 있는 라이브러리를 개발](https://www.lightbend.com/blog/kafka-http-interactive-layer)했는데 현재는 개발이 중지된 상태인 것 같습니다.

Kafka Streams가 궁금하신 분은 [공식 홈페이지의 문서](https://kafka.apache.org/documentation/streams/)를 참고해주십시오. 저는 특히 ["Duality of Streams and Tables"](https://kafka.apache.org/23/documentation/streams/core-concepts#streams-concepts-duality) 부분은 Stream과 Table를 바라보는 Kafka Community의 생각을 알 수 있어서 재밌게 읽었습니다. 마지막에 "Essentially, this duality means that a stream can be viewed as a table, and a table can be viewed as a stream."라고 말하고 있는데 제가 이해한 바로는 이렇습니다. Kafka Streams는 데이터를 처리할 때마다 누적된 값을 출력하기 때문에 Kafka의 Log Compaction 기능을 쓰면 같은 Key를 가진 데이터는 항상 가장 마지막에 업데이트된 데이터만 남게 됩니다. 이 stream은 key에 대해 마지막 값만 남으니 처음부터 끝까지 읽으면 full table scan을 한 것과 다른 것이 없습니다. stream을 처음부터 끝까지 읽어서 필요한 데이터를 제외하고 나머지를 모두 버린다면 filtering을 한 것이 됩니다. 또 table에 새로운 값이 계속 들어오고 이것을 CDC나 Select로 읽는다면 마치 Stream처럼 보입니다. 그렇기 때문에 Stream은 Table처럼, Table은 Stream처럼 보일 수 있다고 표현한 것이 아닐까요.

## 박상원 : Kafka 모니터링을 위한 Metrics 이해

- Kafka Monitoring
  - 현재 상황을 파악하고 이상 상태를 감지하고 대응하는 활동
    - 장애방지, 성능개선
  - 어떻게 상황을 파악할 수 있을까
    - DATADOG, confluent, kafka Manager 등의 툴 사용
  - 내 업무에 최적화된 모니터링을 하려면
    - 다른 Metric과 연계한 새로운 지표 생성
    - 측정된 지표에 따른 Alarm 등의 업무 자동화
  - 어떤 정보를 모니터링 해야 할까
    - 클러스터 안정성
      - 데이터 유실없이 중단 없는 서비스를 할 수 있는가
      - 안정성을 위해 확인해야할 Metric 유형
        - 데이터 처리의 핵심인 Partition이 장애없이 정상적으로 운영되는가
        - 서버의 불필요한 Disk I/O가 발행하진 않는가
        - Swap usage
          - swap이 발생하는 원인
            - kafka 구동시 설정한 heap memory를 초과하는 경우, 데이터를 swap 공간으로 복사하게 되는데 한번 swap 공간으로 이동하면 다시 메모리로 돌아오게 할 수 없음 -> 성능 저하 유발
          - swap 발생 조건 변경
            - vm.swappiness: 메모리에서 swap으로 이동을 언제 할지 결정
            - wm.swappiness = 0 으로 설정하면 메모리에서만 처리하여 kafka 성능이 극대화
    - 메시지 적시성
      - producer에서 consumer로 지연없이 잘 전달되는가
      - 적시성을 위해 확인해야할 Metric 유형
        - producer : 얼마나 빨리/많이 보내고 있는가
        - consumer: 얼마나 빨리/많이 수신하고 있는가
          - 전체 consumer, group, topic 별 구분하여 모니터링
        - broker : producer, consumer 양쪽의 요청을 얼마나 빨리/많이 처리하고 있는가
          - cluster, broker, topic 단위로 구분하여 모니터링
    - Consumer 장애 판단 기준
      - consumer와 broker(coordinator)간의 주기적 신호를 통해 장애 판단
        - session time, poll(데이터 요청) time 을 확인
      - consumer 장애 시 partition 재할당
        - consumer의 신호가 없으면(time out) 장애로 판단하고, partition 재할당
    - 클러스터 확장 시점
      - 확장성을 위해 확인해야 할 Metrics 유형
        - 과도한 부하가 발생하는 구간 체크 ( Network, CPU, Memory)
          - producer - broker 사이의 네트워크 부하
          - consumer의 대기 시간 증가
          - broker의 처리 시간 증가
  - 결론
    - 자신의 운영 시스템에 최적화된 모니터링 지표의 조합이 가장 중요

## 윤도영 : 카카오 datalake 소개

- datalake의 정의
  - 모든 데이터를 저장하는 저장소
  - 별도로 structure data로 변환하지 않고 그대로 저장
- datalake의 목적
  - 저장 후 필요한 곳에 활용하기 위함

- 하지만 현실에서는
  - 어떤 데이터가 저장되는지 모름
  - 어디에 저장되는지 모름
  - 어떤 형태로 저장되는지 모름
  - 저장되는 데이터의 추가/변경 사항을 모름 등의 문제가 발생함

* datalake를 이용해 여러 서비스의 데이터를 연결하고, 이 데이터로 충분한 가치를 만들고 싶음
  * 그러기 위해서는
    1. 데이터 소스의 일반화
    2. 메타 데이터의 관리
    3. 메타 데이터를 활용한 데이터 처리 자동화가 필요함

* 예를 들어, 예전에는 결제 관련 데이터를 활용하고 싶을 때
  1. 담당자에게 데이터의 형태, 내용, 위치 등을 문의
  2. 엑셀 형식으로 데이터 제공의 과정을 거쳤다면,

* 새로운 datalake 서비스를 통해

  1. 레이크에 추가되는 데이터들에 대한 메타 데이터를 자동으로 생성, 검색 인덱스에 추가
  2. 필요한 데이터는 권한신청을 통해 사용(데이터 소스 변경 or 장애 발생시 파악이 용이)
  3. 중요 feature/label을 한 곳에서 생성/관리 하므로, 활용하는 클라이언트에게 일원화 된 데이터 제공 가능

  의 과정을 통해 보다 효율적인 데이터 관리 및 사용이 가능함

### 새로운 datalake 시스템 구성

AWS의 Glue 시스템의 구성을 오픈소스를 가지고 모방한 형태

![img](https://github.com/namjunemy/TIL/blob/master/SeminarAndConference/img/kafka_seoul_2019/kafka_02.png?raw=true)

![img](https://github.com/namjunemy/TIL/blob/master/SeminarAndConference/img/kafka_seoul_2019/kafka_03.png?raw=true)

이 중 핵심 컴포넌트는

1. Apache Kakfa : streaming data source
   - 안정성, 성능이 검증되었기 때문에 데이터 레이크에서 ingest layer를 담당함
   - 가능하면 publish 시점부터 Schema를 registry하는 것이 유리함
2. S2 Catalog
   - Apache Spark의 DataFrame: Schema discrepancies
     - Two pass : InferSchema -> Load the data
     - 같은 field가 record별로 type이 다르면 -> string으로 처리 or corrupted record로 처리
   - 계속해서 쌓이는 DataFrame에 field가 추가되거나, 없어지는 경우
     - 현재 DataFrame과 catalog에 schema 정보를 merge한 후 다시 catalog를 update
   - Incremental하게 데이터 처리시 bookeeping
     - 이전 작업 파일의 변경 시간, DB record의 id, timestamp등을 관리
     - 다시 작업을 실행해도 작동으로 어떤 데이터를 처리해야 할지 판단

참고) 현재 오픈소스화 진행중이라고 합니다