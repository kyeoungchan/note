# 💻 Kafka 커맨드 라인 툴
- [kafka-topic.sh](#-kafka-topicsh)
  - [토픽 생성](#-토픽-생성)
  - [토픽 리스트 조회](#-토픽-리스트-조회)
  - [토픽 옵션 수정](#-토픽-옵션-수정)
- [kafka-console-producer.sh](#-kafka-console-producersh)
- [kafka-console-consumer.sh](#-kafka-console-consumersh)
- [kafka-consumer-groups.sh](#-kafka-consumer-groupssh)
- [kafka-verifiable-producer.sh, kafka-verifiable-consumer.sh](#-kafka-verifiable-producersh-kafka-verifiable-consumersh)
- [kafka-delete-records.sh](#-kafka-delete-recordssh)

## ❗️ kafka-topic.sh
토픽이란, 카프카에서 데이터를 구분하는 가장 기본적인 개념이다.  
마치 RDBMS에서 사용하는 테이블과 유사하다고 볼 수 있다.  
카프카 클러스터에 토픽은 여러 개 존재할 수 있다.  
토픽에는 파티션(partition)이 존재하는데, 파티션의 개수는 최소 1개부터 시작할 수 있다.  
파티션을 통해 한 번에 처리할 수 있는 데이터양을 늘릴 수 있고, 토픽 내부에서도 파티션을 통해 데이터의 종류를 나누어 처리할 수 있다.  

> 토픽을 생성하는 2가지 방법  
> 1. 카프카 컨슈머 또는 프로듀서가 카프카 브로커에 생성되지 않은 토픽에 대해 데이터를 요청할 때
> 2. 커맨드 라인 툴로 명시적으로 토픽을 생성할 때
> 
> 토픽을 효과적으로 유지보수하기 위해서는 토픽을 명시적으로 생성하기를 추천한다.  
> - 동시 데이터 처리량이 많아야 하는 토픽의 경우 파티션의 개수를 100으로 설정할 수 있다.  
> - 단기간 데이터 처리만 필요한 경우에는 토픽에 들어온 데이터의 보관기간 옵션을 짧게 설정할 수 있다.  
> 
> 토픽에 들어오는 데이터양과 병렬로 처리되어야 하는 용량을 잘 파악하여 생성하는 것이 중요하다.

<br>

### ✅ 토픽 생성
```shell
Kyeongchanui-MacBookPro:kafka_2.12-3.9.0 kyeongchanwoo$ bin/kafka-topics.sh \
> --create \
> --bootstrap-server my-kafka:9092 \
> --topic hello.kafka
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic hello.kafka.
```
- `--create`: 토픽을 생성하는 명령어
- `--bootstrap-server`: 토픽을 생성할 카프카 클러스터를 구성하는 브로커들의 IP와 port를 적는다.
- `--topic`: 토픽 이름을 작성한다.
  - 토픽 이름은 내부 데이터가 무엇이 있는지 유추가 가능할 정도로 자세히 적는 것을 추천한다.

모두 브로커에 설정된 기본값으로 파티션 개수, 복제 개수 같은 옵션이 지정됐다.

<br>

```shell
Kyeongchanui-MacBookPro:kafka_2.12-3.9.0 kyeongchanwoo$ bin/kafka-topics.sh \
> --create \
> --bootstrap-server my-kafka:9092 \
> --partitions 3 \
> --replication-factor 1 \
> --config retention.ms=172800000 \
> --topic hello.kafka.2
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic hello.kafka.2.
```
- `--partitions`: 파티션 개수를 지정한다.
  - 파티션 최소 개수는 1개다.
  - 만약 이 옵션을 지정하지 않으면 kafka broker 설정 파일(config/kraft/server.properties)의 `num.partitions` 옵션값에 따라 지정된다.
- `--replication-factor`: 토픽의 파티션을 복제할 복제 개수를 지정한다.
  - 1: 복제를 하지 않고 사용한다는 의미
  - 2: 1개의 복제본을 사용한다는 의미
  - 파티션의 데이터는 각 브로커마다 저장된다.
    - 한 개의 브로커에 장애가 발생하더라도 다른 브로커에 저장된 데이터를 통해 대응할 수 있다.
  - 복제 개수의 최소 설정은 1이고, 최대 설정은 통신하는 카프카 클러스터의 브로커 개수다.
    - 현재 나는 브로커가 하나이므로 설정할 수 있는 값이 1로 유일하다.
- `--config`: `kafka-topics.sh`에 포함되지 않은 추가 설정을 할 수 있다.
  - `retention.ms`: 토픽의 데이터 유지 기간을 뜻한다.
    - 172800000ms = 2일이다.
    - 2일이 지난 데이터는 삭제된다.

<br>

### ✅ 토픽 리스트 조회
```shell
$ bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --list
hello.kafka
hello.kafka.2
```

<br>

### ✅ 토픽 상세 조회
```shell
$ bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --describe --topic hello.kafka.2
Topic: hello.kafka.2	TopicId: Yoqy3dStRKKmz3Kwyny9Kw	PartitionCount: 3	ReplicationFactor: 1	Configs: segment.bytes=1073741824,retention.ms=172800000
	Topic: hello.kafka.2	Partition: 0	Leader: 2	Replicas: 2	Isr: 2	Elr: 	LastKnownElr: 
	Topic: hello.kafka.2	Partition: 1	Leader: 2	Replicas: 2	Isr: 2	Elr: 	LastKnownElr: 
	Topic: hello.kafka.2	Partition: 2	Leader: 2	Replicas: 2	Isr: 2	Elr: 	LastKnownElr: 
```
- `--describe`: 생성된 토픽의 상태를 확인할 수 있다.
- `Leader`: 브로커의 id다.
  - EC2 서버에서 `config/kraft/server.properties`에서 `node.id=2`여서 저렇게 나온다.
  - 여러 대의 브로커로 카프카 클러스터를 운영할 때 리더 파티션이 일부 브로커로 쏠림 현상을 파악할 때 사용한다.

<br>

### ✅ 토픽 옵션 수정
- 파티션 개수 변경 ➡ `kafka-topics.sh`
- 토픽 삭제 정책인 리텐션 기간 변경 ➡ `kafka-configs.sh`

```shell
# 파티션 개수 늘리기
Kyeongchanui-MacBookPro:kafka_2.12-3.9.0 kyeongchanwoo$ bin/kafka-topics.sh --bootstrap-server my-kafka:9092 \
> --topic hello.kafka \
> --alter \
> --partitions 4

# 파티션 생성 결과 확인
Kyeongchanui-MacBookPro:kafka_2.12-3.9.0 kyeongchanwoo$ bin/kafka-topics.sh --bootstrap-server my-kafka:9092 \
> --topic hello.kafka \
> --describe
Topic: hello.kafka	TopicId: uOFV_1f4Soqf4HtjpHYBmQ	PartitionCount: 4	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: hello.kafka	Partition: 0	Leader: 2	Replicas: 2	Isr: 2	Elr: 	LastKnownElr: 
	Topic: hello.kafka	Partition: 1	Leader: 2	Replicas: 2	Isr: 2	Elr: 	LastKnownElr: 
	Topic: hello.kafka	Partition: 2	Leader: 2	Replicas: 2	Isr: 2	Elr: 	LastKnownElr: 
	Topic: hello.kafka	Partition: 3	Leader: 2	Replicas: 2	Isr: 2	Elr: 	LastKnownElr:

# retention.ms 수정
Kyeongchanui-MacBookPro:kafka_2.12-3.9.0 kyeongchanwoo$ bin/kafka-configs.sh --bootstrap-server my-kafka:9092 \
> --entity-type topics \
> --entity-name hello.kafka \
> --alter --add-config retention.ms=86400000
Completed updating config for topic hello.kafka.

# retention.ms 결과 확인
Kyeongchanui-MacBookPro:kafka_2.12-3.9.0 kyeongchanwoo$ bin/kafka-configs.sh --bootstrap-server my-kafka:9092 \
> --entity-type topics \
> --entity-name hello.kafka \
> --describe
Dynamic configs for topic hello.kafka are:
  retention.ms=86400000 sensitive=false synonyms={DYNAMIC_TOPIC_CONFIG:retention.ms=86400000}
```
- `--alter` 옵션과 `--partitions` 옵션을 함께 사용하면 파티션의 개수 변경 가능
  - 토픽의 파티션을 늘릴 수 있지만 줄일 수는 없다.  
    ➡ 파티션 개수를 반드시 늘려야하는 상황인지 판단 필요
- 파티션 번호는 0부터 1씩 늘어난다.
- `--add-config`: 이미 존재하는 설정값은 변경하고, 존재하지 않는 값은 신규로 추가한다.

<br>

## ❗️ kafka-console-producer.sh
생성된 `hello.kafka` 토픽에 데이터를 넣을 수 있는 `kafka-console-producer.sh` 명령어를 실행해보자.  
레코드(record): 토픽에 넣는 데이터로, key와 value 형식으로 이루어져 있다.  
메시지 key 없이 value로만 보내면 key는 `null`로 기본 설정되어 브로커로 전송된다.  
```shell
Kyeongchanui-MacBookPro:kafka_2.12-3.9.0 kyeongchanwoo$ bin/kafka-console-producer.sh --bootstrap-server my-kafka:9092 \
> --topic hello.kafka
>hello
>kafka
>5
>3
>1
```

프로듀서 정상 종료: `CTRL + D`

<br>

`kafka-console-producer.sh`로 전송되는 레코드 값은 UTF-8 기반으로 Byte로 변환되고, `ByteArraySerializer`로만 직렬화된다.  
`String`이 아닌 타입으로는 직렬화하여 전송할 수 없으므로, 다른 타입으로 직렬화하여 데이터를 브로커로 전송하고 싶다면, 카프카 프로듀서 애플리케이션을 직접 개발해야 한다.

<br>

아래는 메시지 key를 가지는 레코드 전송이다.
```shell
Kyeongchanui-MacBookPro:kafka_2.12-3.9.0 kyeongchanwoo$ bin/kafka-console-producer.sh --bootstrap-server my-kafka:9092 \
> --topic hello.kafka \
> --property "parse.key=true" \
> --property "key.separator=:"
>key1:no1
>key2:no2 
>key3:no3
```
- `parse.key=true`로 두면 레코드를 전송할 때 메시지 `key`를 추가할 수 있다.
- `key.separator=:`
  - 메시지 `key`와 `value`를 구분하는 구분자 선언
  - 기본은 Tab delimiter(`\t`)다.
  - 만약 `key.separator`로 사용하는 구분자를 넣지 않고 Enter를 누르면 `KafkaException`과 함께 종료된다.

<br>

파티션 할당
- 메시지 `key`가 `null`
  - 프로듀서가 파티션으로 전송할 때 레코드 배치 단위(레코드 전송 묶음)으로 라운드로빈으로 전송한다.
- 메시지 `key` 존재
  - 디폴트 세팅의 경우, `key`의 해시값을 작성하여 존재하는 파티션 중 한 개에 할당한다.  
    ➡ 메시지 `key`가 동일하면 동일한 파티션으로 간다.

<br>

> `partition` 개수가 늘어나면 새로 프로듀싱되는 레코드들은 어드 파티션으로 가는가?  
> 메시지 `key`를 가진 레코드의 경우, `partition` 추가되면 `partition`과 메시지 `key`의 일관성이 보장되지 않는다.  
> `partition`을 추가하더라도 이전에 사용하던 메시지 `key`의 일관성을 보장하고 싶다면 커스텀 `partitioner`를 만들어서 운영해야 한다.

<br>

## ❗️ kafka-console-consumer.sh
`hello.kafka` 토픽으로 전송된 데이터는 `kafka-console-consumer.sh` 커맨드로 확인할 수 있다.  

<br>

필수 옵션
- `bootstrap-server`: 카프카 클러스터 정보
- `--topic`: 토픽 이름
- `--from-beginning`: 필수는 아니지만, 토픽에 저장된 가장 처음 데이터부터 출력할 수 있다.

```shell
# 렉이 걸려서 메시지 개수를 최대 100으로 설정했다. t2 micro 너무 성능 안 좋다..
Kyeongchanui-MacBookPro:kafka kyeongchanwoo$ bin/kafka-console-consumer.sh \
> --bootstrap-server my-kafka:9092 \
> --topic hello.kafka \
> --from-beginning \
> --max-messages 100
no2 
hello
kafka
5
3
1
no3
no1
```

<br>
 
```shell
# 메시지 키와 메시지 값 확인
Kyeongchanui-MacBookPro:kafka kyeongchanwoo$ bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 \
> --topic hello.kafka \
> --property print.key=true \
> --property key.separator="-" \
> --group hello-group \
> --from-beginning \
> --max-messages 100
key2-no2 
null-hello
null-kafka
null-5
null-3
null-1
key3-no3
key1-no1
```
- `--property print.key`: 메시지 `key`를 확인하기 위한 설정
- `--property key.separator`: `key`와 `value`를 구분하기 위한 구분자 설정
  - 디폴트는 탭(`\t`)이다.
- `--group`: 신규 컨슈머 그룹을 생성했다.
  - 이 컨슈머 그룹을 통해 가져간 토픽의 메시지는 가져간 메시지에 대해 커밋을 한다.
  - 커밋 정보는 `__conusmer_offsets` 이름의 내부 토픽에 저장된다.

<br>

## ❗️ kafka-consumer-groups.sh
`hello-group` 컨슈머 그룹으로 생성된 컨슈머로 `hello.kafka` 토픽의 데이터를 가져갔다.  
컨슈머 그룹은 따로 생성하는 명령을 날리지 않고, 컨슈머를 동작할 때 컨슈머 그룹 이름을 지정하면 새로 생성된다.  
생성된 컨슈머 그룹의 리스트는 `kafka-consumer-groups.sh` 커맨드로 확인할 수 있다.  
```shell
Kyeongchanui-MacBookPro:kafka kyeongchanwoo$ bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 --list
hello-group
```

<br>

```shell
Kyeongchanui-MacBookPro:kafka kyeongchanwoo$ bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 \
> --group hello-group \
> --describe

Consumer group 'hello-group' has no active members.

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
hello-group     hello.kafka     3          1               1               0               -               -               -
hello-group     hello.kafka     2          5               5               0               -               -               -
hello-group     hello.kafka     1          1               1               0               -               -               -
hello-group     hello.kafka     0          1               1               0               -               -               -
```
- `TOPIC`: `hello-group`이 마지막으로 조회한 토픽
- `PARTITION`: `hello-group`이 마지막으로 조회한 파티션
- `CURRENT-OFFSET`: `hello-group`이 가져간 토픽의 파티션에 가장 최신 오프셋이 몇 번인지 나타낸다.
  - 3번 파티션은 1이 오프셋으로, 1개의 데이터가 들어갔고, 2번 파티션은 5번 오프셋으로, 5개의 데이터가 들어갔다.
- `LOG-END-OFFSET`: 전체 데이터의 총량으로, `CURRENT-OFFSET`는 `LOG-END-OFFSET`보다 작거나 같을 수 있다.
- `LAG`: 파티션에 있는 데이터를 가져가는 데 얼마나 지연이 발생했는지 나타내는 지표다.
  - 커밋한 오프셋과 파티션의 가장 최신 오프셋의 차이를 나타낸다.
  - 둘이 같으므로 0으로 돼있다.

<br>

## ❗️ kafka-verifiable-producer.sh, kafka-verifiable-consumer.sh
`kafka-verifiable`로 시작하는 2개의 스크립트를 사용하면 String 타입 메시지 값을 코드 없이 주고받을 수 있다.  
카프카 클러스터가 설치 완료 후 토픽에 데이터를 전송하여 간단한 네트워크 통신 테스트를할 때 유용하다.  

```shell
Kyeongchanui-MacBookPro:kafka kyeongchanwoo$ bin/kafka-verifiable-producer.sh --bootstrap-server my-kafka:9092 \
> --max-messages 10 \
> --topic verify-test
{"timestamp":1764489857188,"name":"startup_complete"}
[2025-11-30 17:04:17,352] WARN [Producer clientId=producer-1] The metadata response from the cluster reported a recoverable issue with correlation id 1 : {verify-test=UNKNOWN_TOPIC_OR_PARTITION} (org.apache.kafka.clients.NetworkClient)
{"timestamp":1764489857585,"name":"producer_send_success","key":null,"value":"0","offset":0,"partition":0,"topic":"verify-test"}
{"timestamp":1764489857587,"name":"producer_send_success","key":null,"value":"1","offset":1,"partition":0,"topic":"verify-test"}
{"timestamp":1764489857587,"name":"producer_send_success","key":null,"value":"2","offset":2,"partition":0,"topic":"verify-test"}
{"timestamp":1764489857587,"name":"producer_send_success","key":null,"value":"3","offset":3,"partition":0,"topic":"verify-test"}
{"timestamp":1764489857587,"name":"producer_send_success","key":null,"value":"4","offset":4,"partition":0,"topic":"verify-test"}
{"timestamp":1764489857588,"name":"producer_send_success","key":null,"value":"5","offset":5,"partition":0,"topic":"verify-test"}
{"timestamp":1764489857588,"name":"producer_send_success","key":null,"value":"6","offset":6,"partition":0,"topic":"verify-test"}
{"timestamp":1764489857588,"name":"producer_send_success","key":null,"value":"7","offset":7,"partition":0,"topic":"verify-test"}
{"timestamp":1764489857588,"name":"producer_send_success","key":null,"value":"8","offset":8,"partition":0,"topic":"verify-test"}
{"timestamp":1764489857588,"name":"producer_send_success","key":null,"value":"9","offset":9,"partition":0,"topic":"verify-test"}
{"timestamp":1764489857596,"name":"shutdown_complete"}
{"timestamp":1764489857598,"name":"tool_data","sent":10,"acked":10,"target_throughput":-1,"avg_throughput":24.390243902439025}
```
- `--max-messages`: `kafka-verifiable-producer.sh`로 보내는 데이터 개수를 지정한다.
  - 만약 `-1`을 입력하면 `kafka-verifiable-producer.sh`가 종료될 때까지 계속 데이터를 토픽으로 보낸다.
- `--topic verify-test`: 데이터를 받을 대상 토픽을 입력한다.
- 메시지 별로 보낸 시간과 메시지 `key`, `value`, `topic`, `partition`, `offset` 번호가 출력된다.
- 마지막에는 통계값이 출력된다. ➡ 평균 처리량을 확인할 수 있다.

<br>

```shell
Kyeongchanui-MacBookPro:kafka kyeongchanwoo$ bin/kafka-verifiable-consumer.sh --bootstrap-server my-kafka:9092 \
> --topic verify-test \
> --group-id test-group
{"timestamp":1764490093945,"name":"startup_complete"}
{"timestamp":1764490097317,"name":"partitions_assigned","partitions":[{"topic":"verify-test","partition":0}]}
{"timestamp":1764490097492,"name":"records_consumed","count":10,"partitions":[{"topic":"verify-test","partition":0,"count":10,"minOffset":0,"maxOffset":9}]}
{"timestamp":1764490097537,"name":"offsets_committed","offsets":[{"topic":"verify-test","partition":0,"offset":10}],"success":true}
```
- `--topic verify-test`: 데이터를 가져오고자 하는 토픽을 지정한다.
- `--group-id test-group`: 컨슈머 그룹을 지정한다.
- 컨슈머는 한 번에 다수의 메시지를 가져와서 처리하므로 한 번에 10개의 메시지를 정상적으로 받았음을 알 수 있다.  
  메시지 수신 이후 10번 오프셋 커밋 여부도 확인할 수 있다.

<br>

## ❗️ kafka-delete-records.sh
이미 적재된 토픽의 데이터를 지우는 방법으로 `kafka-delete-records.sh`를 사용할 수 있다.  
`kafka-delete-records.sh`는 이미 적재된 토픽의 데이터 중 가장 오래된 데이터(가장 낮은 숫자의 오프셋)부터 특정 시점의 오프셋까지 삭제할 수 있다.  



<br>

```shell
Kyeongchanui-MacBookPro:kafka kyeongchanwoo$ vi delete-topic.json

# delete-topic.json에 새로 입력
{"partitions": [{"topic": "test", "partition": 0, "offset": 50}], "version": 1}
```
```shell
Kyeongchanui-MacBookPro:kafka kyeongchanwoo$ bin/kafka-delete-records.sh --bootstrap-server my-kafka:9092 \
> --offset-json-file delete-topic.json
Executing records delete operation
Records delete operation completed:
```
- `delete-topic.json`: 삭제하고자 하는 데이터에 대한 정보를 파일로 저장해서 사용해야 한다.
  - 해당 파일에는 삭제하고자 하는 토픽, 파티션, 오프셋에 대한 정보가 있어야 한다.
- `--offset-json-file delete-topic.json`: 삭제 정보를 다음은 파일을 `--offset-json-file` 옵션 값으로 입력하여 데이터 삭제를 진행한다. 






<br>


**참고 자료**  
[아파치 카프카 애플리케이션 프로그래밍 with 자바](https://product.kyobobook.co.kr/detail/S000001842177)