# 💻 카프카 기본 개념
- [카프카 브로커・클러스터・주키퍼](#-카프카-브로커클러스터주키퍼)
  - [데이터 저장, 전송](#-데이터-저장-전송)
  - [데이터 복제, 싱크](#-데이터-복제-싱크)
  - [컨트롤러(controller)](#-컨트롤러controller)
  - [데이터 삭제](#-데이터-삭제)
  - [컨슈머 오프셋 저장](#-컨슈머-오프셋-저장)
  - [코디네이터(coordinator)](#-코디네이터coordinator)
- [토픽과 파티션](#-토픽과-파티션)
  - [토픽 이름 제약 조건](#-토픽-이름-제약-조건)

> 나는 kraft 모드로 주키퍼는 사용하지 않지만, 기본 구조를 파악해야 적용이 가능하므로 주키퍼에 대한 내용도 다룬다.

## ❗️ 카프카 브로커・클러스터・주키퍼
![zookeeper_kafka_cluster.jpeg](../res/zookeeper_kafka_cluster.jpeg)  

- 카프카 브로커는 카프카 클라이언트와 데이터를 주고받기 위해 사용하는 주체이자, 데이터를 분산 저장하여 장애가 발생해도 안전하게 사용할 수 있도록 도와주는 애플리케이션이다.
- 하나의 서버에는 한 개의 카프카 브로커 프로세스가 실행된다.
- 데이터를 안전하게 보관하고 처리하기 위해 3대 이상의 브로커 서버를 1개의 클러스터로 묶어서 운영한다.
- 카프카 클러스터로 묶인 브로커들은 프로듀서가 보낸 데이터를 안전하게 분산 저장하고 복제하는 역할을 한다.

<br>

### ✅ 데이터 저장, 전송
프로듀서로부터 전달된 데이터는 파일 시스템에 저장된다.  

```shell
# EC2 환경
$ ls /tmp/kraft-combined-logs
...
__consumer_offsets-10  __consumer_offsets-2   __consumer_offsets-29  __consumer_offsets-38  __consumer_offsets-47  hello.kafka-0              replication-offset-checkpoint
__consumer_offsets-11  __consumer_offsets-20  __consumer_offsets-3   __consumer_offsets-39  __consumer_offsets-48  hello.kafka-1              verify-test-0
__consumer_offsets-12  __consumer_offsets-21  __consumer_offsets-30  __consumer_offsets-4   __consumer_offsets-49  hello.kafka-2
__consumer_offsets-13  __consumer_offsets-22  __consumer_offsets-31  __consumer_offsets-40  __consumer_offsets-5   hello.kafka-3
__consumer_offsets-14  __consumer_offsets-23  __consumer_offsets-32  __consumer_offsets-41  __consumer_offsets-6   hello.kafka.2-0
__consumer_offsets-15  __consumer_offsets-24  __consumer_offsets-33  __consumer_offsets-42  __consumer_offsets-7   hello.kafka.2-1
__consumer_offsets-16  __consumer_offsets-25  __consumer_offsets-34  __consumer_offsets-43  __consumer_offsets-8   hello.kafka.2-2
```
- `config/kraft/server.properties`의 `log.dir` 옵션에 정의한 디렉토리에 데이터가 존재한다.
  - `hello.kafka`는 4개의 파티션이 있어, 4개의 디렉토리가 존재한다.

<br>

```shell
# EC2 환경
$ ls /tmp/kraft-combined-logs/hello.kafka-0
00000000000000000000.index  00000000000000000000.log  00000000000000000000.timeindex  00000000000000000001.snapshot  leader-epoch-checkpoint  partition.metadata
```
- `hello.kafka` 0번 파티션에 존재하는 데이터를 확인할 수 있다.  
- `log`에는 메시지와 메타데이터를 저장한다.
- `index`에는 메시지의 오프셋을 인덱싱한 정보를 저장한다.
- `timeindex`에는 메시지에 포함된 `timestamp`값을 기준으로 한 인덱싱한 정보를 저장한다.

<br>

### ✅ 데이터 복제, 싱크
데이터 복제(replication)는 카프카를 장애 허용 시스템(fault tolerant system)으로 동작하도록 하는 원동력이다.  
카프카의 데이터 복제는 파티션 단위로 이루어진다.  
➡ 토픽을 생성할 때 파티션의 복제 개수(replication factor)도 같이 설정된다.  
➡ 직접 옵션을 선택하지 않으면 브로커에 설정된 옵션 값을 따라간다.  
복제 개수의 최솟값은 1(복제 없음)이고, 최댓값은 브로커 개수만큼 설정할 수 있다.

<br>

![partition_replication_3.jpeg](../res/partition_replication_3.jpeg)  
- 위 그림은 복제 개수가 3인 경우다.
- 리더: 프로듀서 또는 컨슈머와 직접 통신하는 파티션
- 팔로워: 나머지 복제 데이터를 가지고 있는 파티션
  - 리더의 오프셋과 자신의 오프셋을 비교해 차이가 나는 경우 리더로부터 데이터를 복제해온다.
- 복제 개수만큼 저장 용량이 증가한다는 단점이 있다.
- 그러나 복제를 통해 데이터를 안전하게 사용할 수 있다는 강력한 장점이 있기 때문에 운영 환경에서 2 이상의 복제 개수를 정하는 것이 중요하다.
  - 데이터가 일부 유실되어도 무관하고 데이터 처리 속도가 중요하다면 1 또는 2로 설정한다.
  - 금융권과 같이 유실이 일어나면 안 되는 데이터의 경우 복제 개수를 3으로 설정한다.   
    ➡ 최대 2대의 브로커가 동시에 장애가 발생하더라도 데이터를 안정적으로 유지할 수 있다.

<br>

### ✅ 컨트롤러(controller)
컨트롤러는 다른 브로커들의 상태를 체크하고 브로커가 클러스터에서 빠지는 경우 해당 브로커에 존재하는 리더 파티션을 재분배한다.  
카프카는 실시간 데이터 처리가 핵심이므로 브로커의 상태가 비정상이라면 빠르게 클러스터에서 빼내는 것이 중요하다.  
만약 컨트롤러 역할을 하는 브로커에 장애가 발생한다면 다른 브로커가 컨트롤러 역할을 한다.

<br>

**KRaft 컨트롤러(Controller Quorum)**  
KRaft 컨트롤러의 경우, 기존 컨트롤러보다 책임 범위가 훨씬 크고, 구조도 다르다.  
- 브로커들 중 하나가 controller 역할을 하지 않고, 별도 `Controller Quorum` 노드가 존재한다.
- 메타 데이터를 자체 KRaft 로그로 저장한다.
- 리더 브로커 장애 발생시 Raft 합의 알고리즘 기반으로 리더를 선출한다.(KRaft = Kafka + Raft)
- 기존은 파티션 리더 재할당, 브로커 상태 모니터링만 했지만, KRaft 컨트롤러는 카프카 전체 메타데이터를 관리한다.
  - 토픽
  - 파티션
  - ACL(Access Control List; 접근 제어 목록)


<br>

### ✅ 데이터 삭제
카프카는 다른 메시징 플랫폼과 달리 컨슈머가 데이터를 가져가더라도 토픽의 데이터는 삭제되지 않는다.  
컨슈머나 프로듀서가 데이터 삭제를 요청할 수도 없다.  
오직 브로커만이 데이터를 삭제할 수 있다.  

<br>

데이터 삭제가 이루어지는 단위: 로그 세그먼트(log segment)  
➡ 파일 단위로 이루어진다.  
하나의 세그먼트에 다수의 데이터가 들어 있기 때문에 RDBMS처럼 특정 데이터를 선정해서 삭제할 수 없다.  
- 세그먼트는 데이터가 쌓이는 동안 파일 시스템으로 열려있으며, 카프카 브로커에 `log.segment.bytes` 또는 `log.segment.ms` 옵션 값에 따라서 세그먼트트 파일이 닫힌다.  
  - 디폴트는 1GB 용량에 도달했을 때 닫히게 되어있다.
  - 너무 작은 용량으로 설정하면 데이터들을 저장하는 동안 파일을 너무 자주 여닫음으로써 부하가 발생할 수 있으므로 주의가 필요하다.
- 닫힌 세그먼트 파일은 `log.retention.bytes` 또는 `log.retention.ms` 설정값이 넘으면 삭제된다.
- `log.retention.check.interval.ms` 따라 닫힌 세그먼트 파일을 체크하는 간격이 설정된다.

<br>

### ✅ 컨슈머 오프셋 저장
컨슈머 그룹은 토픽이 특정 파티션으로부터 데이터를 가져가고 처리하고 이 파티션의 어느 레코드까지 가져갔는지 확인하기 위해 오프셋을 커밋한다.  
커밋한 오프셋은 `__consumer_offsets` 토픽에 저장한다.  
해당 오프셋을 토대로 컨슈머 그룹은 다음 레코드를 가져가서 처리한다.  

<br>

### ✅ 코디네이터(coordinator)
클러스터의 다수 브로커 중 한 대는 코디네이터의 역할을 수행한다.  
코디네이터는 컨슈머 그룹의 상태를 체크하고, 파티션을 컨슈머와 매칭하도록 분배하는 역할을 수행한다. (리밸런스; `rebalance`)  


<br>

## ❗️ 토픽과 파티션
- 토픽: 카프카에서 데이터를 구분하기 위해 사용하는 단위
  - 토픽에 1개 이상의 파티션 소유
- 파티션: 프로듀서가 보낸 데이터들이 들어가 저장됨
- 레코드: 파티션에 저장된 데이터

<br>

파티션은 큐와 비슷한 구조지만, 큐는 데이터를 가져가면 레코드를 삭제하는 반면, 카프켕서는 삭제하지 않는다.  
➡ 여러 컨슈머 그룹들이 토픽의 데이터를 여러 번 가져갈 수 있다.

<br>

### ✅ 토픽 이름 제약 조건
- 빈 문자열 X
- `.` 또는 `..` X
- 249자 미만으로 생성되어야 한다.
- 영어 대소문자와 숫자, `.`, `_`, `-` 조합으로 생성할 수 있다.  
  그 외의 문자열이 포함된 토픽 이름은 생성 불가
- 카프카 내부 로직 관리 목적으로 사용되는 토픽(`__consumer_offsets`, `__transaction_state`, `__cluster_metadata`)와 동일한 이름으로는 생성 불가
- 카프카 내부 로직 때문에 `.`와 `_`가 동시에 들어가면 안 된다.
  - 생성은 할 수 있지만 이슈가 발생할 수 있기 때문에 WARNING 메시지가 뜬다.
- 이미 생성된 토픽의 이름과 `.` ➡ `_` 또는 `_` ➡ `.` 로 바뀐 거 외에 동일한 이름으로는 생성할 수 없다.

<br>

토픽 작명의 템플릿과 예시
- <환경>.<팀-명>.<애플리케이션-명>.<메시지-타입>
  - `prd.marketing-team.sms-platform.json`
- <프로젝트-명>.<서비스-명>.<환경>.<이벤트-명>
  - `commerce.payment.prd.notification`
- <환경>.<서비스-명>.<JIRA-번호>.<메시지-타입>
  - `dev.email-sender.jira-1234.email-vo-custom`
- <카프카-클러스터-명>.<환경>.<서비스-명>.<메시지-타입>
  - `aws-kafka.live.marketing-platform.json`

<br>

카프카는 토픽 이름 변경을 지원하지 않으므로 이름을 변경하기 위해서는 삭제 후 다시 생성하는 것 외에는 방법이 없다.  
➡ 다로 까다롭더라도 카프카 클러스터 사용자에게 토픽 생성에 대한 규칙을 인지시키는 것이 중요하다.  
➡ 규칙을 따르지 않은 토픽의 이름이 있을 경우, 실제 사용할 토픽이라면 삭제 후 신규로 토픽을 만드는 것을 권장한다.



<br>

**참고 자료**  
[아파치 카프카 애플리케이션 프로그래밍 with 자바](https://product.kyobobook.co.kr/detail/S000001842177)