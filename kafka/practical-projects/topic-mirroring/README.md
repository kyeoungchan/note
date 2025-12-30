# 🧑🏻‍💻 토픽 미러링
- [요구 사항](#-요구-사항)
- [기능 정의](#-기능-정의)
- [기능 구현](#-기능-구현)
- [기능 테스트](#-기능-테스트)
### ✅ 요구 사항
- 2개의 카프카 클러스터가 존재한다는 가정하에 진행한다.  
- 각각의 클러스터를 클러스터A, 클러스터B라고 명명한다.  
- 클러스터A에는 `weather.seoul`이라는 토픽이 있다.
- `weather.seoul` 토픽을 클러스터B에서 사용하기 위해서는 클러스터A의 대상 토픽의 데이터를 복사해야 한다.

특정 클러스터에 존재하는 토픽 데이터를 실시간으로 미러링하는 데에는 미러메이커2가 유용하다. ([미러메이커2 참고](https://github.com/kyeoungchan/note/tree/main/kafka/kafkamirrormaker2))  
➡ 데이터를 옮기는 데에 그치는 것이 아니라 토픽의 파티션 개수 감지, 새로운 토픽 감지, 토픽 설정 변화 감지 등의 기능도 포함되어 있기 때문이다.

<br>

### ✅ 기능 정의
- 각 클러스터의 보안: 클러스터에 보안 설정이 되어 있을 경우 권한을 획득하고 접근할 수 있도록 설정이 필요하다.
- 미러링 토픽 이름: 정규식으로 토픽을 표현할 수 있으므로 미러링해야 하는 토픽이 많을 경우 정규식으로 선언하면 미러메이커2를 재시작할 필요 없이 토픽이 생성되는 대로 모두 미러링할 수 있다.

> 여기서는 클러스터A는 AWS에 구축한 카프카 클러스터, 클러스터B는 로컬 카프카 클러스터로 제한하고, 두 개의 클러스터에는 보안 설정이 되어있지 않다고 가정하고 진행한다.  
> 미러링 토픽은 `weather.seoul`로 한정한다.

<br>

### ✅ 기능 구현
미러메이커2의 기본 설정파일은 카프카 바이너리가 존재하는 디렉토리에서 config 폴더에 `connect-mirror-maker.properties` 파일로 존재한다.  
해당 설정 파일을 다음과 같이 수정한다.
```properties
# 클러스터의 닉네임을 지정한다.
# 실제 운영환경에서 미러링을 진행할 경우 명확히 구분할 수 있는 닉네임을 붙이는 것이 좋다.
clusters = A, B

A.bootstrap.servers = my-kafka:9092
B.bootstrap.servers = localhost:9092

# 클러스터A에서 클러스터B로 미러링하는지 여부와 토픽 이름을 지정한다.
A->B.enabled = true
A->B.topics = weather.seoul

# 클러스터B에서 클러스터A로 미러링하는지 여부와 토픽 이름을 지정한다.
B->A.enabled = false
B->A.topics = .*

# 신규로 생성되는 토픽의 복제 개수를 지정한다.
replication.factor=1

# 미러메이커2를 운영하는 데에 필요한 내부 토픽들의 복제 개수를 설정한다.
# 실제 운영환경에서는 브로커에 장애가 발생할 경우를 대비하여 내부 토픽의 복제 개수는 3 이상으로 설정하는 것이 좋다.
checkpoints.topic.replication.factor=1
heartbeats.topic.replication.factor=1
offset-syncs.topic.replication.factor=1

offset.storage.replication.factor=1
status.storage.replication.factor=1
config.storage.replication.factor=1
```

<br>

### ✅ 기능 테스트
```shell
$ brew update

# 로컬 카프카 설치
$ brew install kafka

# 로컬 카프카 실행
$ brew services start kafka

# 로컬 카프카 종료
$ brew services stop kafka
```
```shell
# 클러스터A에 `weather.seoul` 생성
$ bin/kafka-topics.sh --create \
--bootstrap-server my-kafka:9092 \
--partitions 3 \
--topic weather.seoul
```

```shell
# 미러메이커2 실행
# 미러링하는 토픽 정보를 담은 connect-mirror-maker.properties 파일과 함께 실행하면 미러메이커2가 실행된다.
$ bin/connect-mirror-maker.sh config/connect-mirror-maker.properties
```

```shell
# 로컬 카프카 브로커에 A.weather.seoul 생성 확인
$ bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
A.checkpoints.internal
A.weather.seoul
__consumer_offsets
first_topic
heartbeats
mm2-configs.A.internal
mm2-offsets.A.internal
mm2-status.A.internal
```

```shell
# 클러스터 A의 weather.seoul과 파티션 개수가 동일하다.
$ bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --topic weather.seoul --describe
Topic: weather.seoul	TopicId: HSsgrajNQ7mu1D9Nt__xwQ	PartitionCount: 3	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: weather.seoul	Partition: 0	Leader: 1	Replicas: 1	Isr: 1	Elr: 	LastKnownElr: 
	Topic: weather.seoul	Partition: 1	Leader: 1	Replicas: 1	Isr: 1	Elr: 	LastKnownElr: 
	Topic: weather.seoul	Partition: 2	Leader: 1	Replicas: 1	Isr: 1	Elr: 	LastKnownElr: 

$ bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic A.weather.seoul --describe
Topic: A.weather.seoul	TopicId: IQONokBVR0qZoJv4oHWR_A	PartitionCount: 3	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: A.weather.seoul	Partition: 0	Leader: 1	Replicas: 1	Isr: 1	Elr: 	LastKnownElr: 
	Topic: A.weather.seoul	Partition: 1	Leader: 1	Replicas: 1	Isr: 1	Elr: 	LastKnownElr: 
	Topic: A.weather.seoul	Partition: 2	Leader: 1	Replicas: 1	Isr: 1	Elr: 	LastKnownElr: 
```

<br>

````shell
# 클러스터 A로 데이터를 보내보자.
bin/kafka-console-producer.sh --bootstrap-server my-kafka:9092 --topic weather.seoul
>sunny
>cloudy
````
```shell
# 클러스터 B로 데이터를 조회해보자.
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic A.weather.seoul --from-beginning
sunny
cloudy
```

<br>

### ✅ 상용 인프라 아키텍처
- 미러메이커2: 2개 이상의 서버

이 아키텍처에서는 미러메이커2를 실행하는 서버를 2대 배치하고 카프카 클러스터의 브로커 개수를 3개로 운영하여 일부 서버에 이슈가 발생하더라도 안전하게 토픽을 미러링할 수 있다.  
분산 모드 커넥트를 운영 중이라면 별도로 미러메이커2를 위한 서버를 구축하지 않아도 된다.  
➡ 미러메이커2는 커넥터로도 동작하도록 만들어졌기 때문이다.



<br>

**참고 자료**  
[아파치 카프카 애플리케이션 프로그래밍 with 자바](https://product.kyobobook.co.kr/detail/S000001842177)