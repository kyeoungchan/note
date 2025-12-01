# 💻 레디스를 메시지 브로커로
- [메시징 큐와 이벤트 스트림](#-메시징-큐와-이벤트-스트림-)
  - [레디스를 메시지 브로커로 사용하기](#-레디스를-메시지-브로커로-사용하기)
- [레디스의 pub/sub](#-레디스의-pubsub)
  - [메시지 publish하기](#-메시지-publish하기)
  - [메시지 subscribe하기](#-메시지-subscribe하기)
  - [클러스터 구조에서의 pub/sub](#-클러스터-구조에서의-pubsub)
  - [sharded pub/sub](#-sharded-pubsub)
- [레디스의 list를 메시징 큐로 사용하기](#-레디스의-list를-메시징-큐로-사용하기)
  - [list의 블로킹 기능](#-list의-블로킹-기능)
  - [list를 이용한 원형 큐](#-list를-이용한-원형-큐)
- [Stream](#-stream)
  - [레디스의 `stream`과 아파치 카프카](#-레디스의-stream과-아파치-카프카)
  - [스트림이란?](#-스트림이란)
  - [데이터의 저장](#-데이터의-저장)
  - [스트림 생성과 데이터 입력](#-스트림-생성과-데이터-입력)
  - [데이터의 조회](#-데이터의-조회)
  - [소비자와 소비자 그룹](#-소비자와-소비자-그룹)
  - [ACK와 보류 리스트](#-ack와-보류-리스트)
  - [메시지의 재할당](#-메시지의-재할당)
  - [stream 상태 확인](#-stream-상태-확인)

<br>

**✍🏻 메시지 브로커**  
최근의 서비스 아키텍처는 여러 모듈이 서로 느슨하고 적절하게 연결시킨 구조를 선호하고 있다.  
모듈 간 서로 탄탄한 상호 작용이 필요하기 때문에 효율적인 메시징 솔루션, 즉 메시지 브로커를 필요로 한다.  

서비스 간 커넥션이 실패하는 상황은 언제나 발생할 수 있다.   
➡ 모듈 간의 통신에서는 되도록 비동기(async) 통신을 사용하거나, 동기(sync) 통신의 횟수를 최대한 줄이는 것이 바람직하다.  
당장 메시지를 처리하지 못하더라도 보낸 메시지를 어딘가에 쌓아둔 뒤 나중에 처리할 수 있는 채널을 만들어주는 것이 바로 **메시지 브로커의 핵심 역할**이다.

<br>

## 💡 메시징 큐와 이벤트 스트림 
|               기준                | 메시징 큐                                                    | 이벤트 스트림                                                       |
|:-------------------------------:|----------------------------------------------------------|---------------------------------------------------------------|
|               용어                | 생산자(producer): 데이터를 생성하는 쪽 <br> 소비자(consumer): 데이터를 수신하는 쪽 | 발행자(publisher): 데이터를 생성하는 쪽 <br> 구독자(subscriber): 데이터를 수신하는 쪽 |
|               방향성               |생산자는 소비자의 큐로 데이터를 직접 푸시한다. <br> 2개의 서비스에 같은 메시지를 보내야 할 때 생산자는 2개의 각각 다른 메시징 큐에 각각 데이터를 푸시해야 한다.|발행자는 스트림의 특정 저장소에 하나의 메시지를 보낼 수 있고, 구독자들은 스트림에서 같은 메시지를 pull해갈 수 있기 때문에 메시지를 복제해서 저장하지 않아도 된다.|
|            데이터의 영속성             |소비자가 데이터를 읽어갈 때 큐에서 데이터를 삭제한다.|구독자가 읽어간 데이터는 바로 삭제되지 않고, 저장소의 설정에 따라 특정 기간 동안 저장될 수 있다.|
| 히스토리(메시지를 보내는 도중 새로운 소비자가 추가되면) |소비자는 새롭게 추가된 이후의 이벤트만 확인할 수 있다.|스트림에 쌓인 데이터는 일정기간 지워지지 않기 때문에 새로 추가된 서비스도 이전 데이터의 히스토리를 볼 수 있다.| 
|적합성|일대일 상황에서 유리하다.|다대다 상황에서 유리하다.|

<br>

### ✅ 레디스를 메시지 브로커로 사용하기
**`pub/sub`**  
레디스에서 제공하는 `pub/sub`를 사용하면 빠르고 간단한 방식으로 메시지를 전달할 수 있다.  
레디스의 `pub/sub`에서 모든 데이터는 한 번 채널 전체에 전파된 뒤 삭제되는 일회성의 특징을 가지며, 메시지가 잘 전달됐는지 등의 정보는 보장하지 않는다.  
➡ `fire-and-forget` 패턴이 필요한 간단한 알림 서비스에서는 유용하게 사용될 수 있다.  

> `fire-and-forget`  
> 비동기 프로그래밍에서 사용되는 디자인 패턴으로, 어떤 작업을 수행하고 그 결과에 대한 응답을 기다리지 않고 바로 다음 코드를 실행하는 것을 말한다.  
> 로깅, 이벤트 발행, 통계 데이터 수집과 같이 작업의 성공 또는 실패에 대한 관심이 없는 경우에 활용될 수 있다.  
> 신뢰성이 필요한 경우에는 사용하지 않아야 한다.

<br>

**`list` 자료 구조**  
레디스의 `list` 자료 구조는 메시징 큐로 사용하기에 알맞다.  
push와 pop이 가능하며, 대기하다가 `list`에 새로운 데이터가 들어오면 읽어갈 수 있는 블로킹 기능을 사용할 수도 있다.  

<br>

**`stream` 자료 구조**  
레디스를 완벽한 스트림 플랫폼으로 사용할 수 있다.  
레디스 `stream`은 아파치 [`kafka` 시스템](https://github.com/kyeoungchan/note/tree/main/kafka)에서 영감을 받아 만들어진 자료 구조로, 데이터는 계속해서 추가되는 방식으로 저장된다(append-only).  
소비자와 소비자 그룹이라는 개념을 이용하면 카프카에서와 비슷하게 데이터의 분산 처리를 구현할 수 있다.  
`stream`에 저장되는 메시지를 실시간으로 리스닝하면서 소비할 수 있고, 저장돼있는 데이터를 시간대별로 검색하는 것도 가능하다.  

<br>

## 💡 레디스의 `pub/sub`
![pub_sub.jpeg](../res/pub_sub.jpeg)  
레디스 노드에 접근할 수 있는 모든 클라이언트는 발행자와 구독자가 될 수 있다.  
발행자는 특정 채널에 메시지를 보낼 수 있으며, 구독자는 특정 채널을 리스닝하다가 메시지를 읽어갈 수 있다.  

위에서 말했다시피, 매우 가볍기 때문에 최소한의 메시지 전달 기능만 제공한다.
- 발행자는 어떤 구독자가 메시지를 읽어가는지, 정상적으로 모든 구독자에게 메시지가 전달됐는지 확인 할 수 없다.
- 구독자 또한 해당 메시지가 언제 어떤 발행자에 의해 생성됐는지 등의 메타데이터는 알 수 없다.

정합성이 중요한 데이터를 전달하기에는 적합하지 않으며, 애플리케이션 레벨에서 메시지의 송수신과 관련한 로직을 추가해야할 수 있다.

### ✅ 메시지 publish하기
```shell
127.0.0.1:6379> PUBLISH hello world
0
```
위의 커맨드를 수행하면 `hello`라는 채널을 수신하고 있는 모든 서버들에 `world`라는 메시지가 전파된다.  
메시지가 전파된 후에는 메시지를 수신한 구독자의 수가 반환된다.  

<br>

### ✅ 메시지 subscribe하기
```shell
127.0.0.1:6379> SUBSCRIBE event1 event2
subscribe
event1
1
subscribe
event2
2
```
클라이언트가 위의 커맨드를 수행하면 event1과 event2 채널을 동시에 구독하기 시작한다.  
클라이언트가 구독자로 동작할 때에는 새로운 채널을 구독할 수 있지만 `pub/sub`과 관련되지 않은 다른 커맨드는 수행할 수 없다.  

<br>

ssh 창을 2개를 띄워서, 하나는 발행, 하나는 구독해보자.  
같은 redis 파일을 2번 실행시키면 된다.
```shell
# 발행자 클라이언트
127.0.0.1:6379> PUBLISH event1 hello
1
127.0.0.1:6379> PUBLISH event1 hello2
1
127.0.0.1:6379> PUBLISH event2 hello3
1
```
```shell
# 구독자 클라이언트
127.0.0.1:6379> SUBSCRIBE event1 event2
subscribe
event1
1
subscribe
event2
2

# hello 메시지 수신
message
event1
hello

# hello2 메시지 수신 
message
event1
hello2

# hello3 메시지 수신
message
event2
hello3
```

<br>

`PSUBSCRIBE` 커맨드를 사용하면 일치하는 패턴에 해당하는 채널을 한 번에 구독할 수 있다.  
이때 레디스는 glob-style 패턴을 지원한다.  
이때 메시지는 `message`가 아닌 `pmessage` 타입으로 전달된다.  
```shell
# 발행자 클라이언트
127.0.0.1:6379> PUBLISH mail-1 hello~!
1
```
```shell
# 구독자 클라이언트
127.0.0.1:6379> PSUBSCRIBE mail-*
psubscribe
mail-*
1
pmessage
mail-*
mail-1
hello~!
```
만약 구독자가 `SUBSCRIBE mail-1`과 `PSUBSCRIBE mail-*`을 동시에 구독하고 있을 때 `mail-1` 채널에 메시지가 발행되면 구독자는 2개의 메시지를 받게 된다.  

<br>

### ✅ 클러스터 구조에서의 pub/sub
레디스 클러스터 구조에서도 `pub/sub`을 사용할 수 있다.  
클러스터는 레디스가 자체적으로 제공하는 데이터 분산 형태의 구조다.  
레디스 클러스터에서 `pub/sub`을 사용할 때, 메시지를 발행하면 해당 메시지는 클러스터에 속한 모든 노드에 자동으로 전달된다.  
따라서 레디스 클러스터의 아무 노드에 연결해 `SUBSCRIBE` 커맨드를 사용하면 데이터를 수신할 수 있다.  

![cluster_pub_sub.jpeg](../res/cluster_pub_sub.jpeg)  

<br>

**클러스터의 목적에는 부합하지 않는 방식**  
클러스터는 주로 대규모 서비스에서 데이터를 분산해서 저장하고 처리하기 위해 도입됐다.  
➡ 레디스 클러스터 내에서 `pub/sub`을 사용할 때 메시지가 모든 레디스 노드에 복제되는 방식은 클러스터 환경의 핵심 목표와는 부합하지 않으며, 이로 인해 불필요한 리소스 사용과 네트워크 부하가 발생할 수 있다.

<br>

### ✅ sharded pub/sub
레디스 7.0에서는 `sharded pub/sub` 기능이 도입됐다.  
`sharded pub/sub` 환경에서 각 채널은 슬롯에 매핑된다.  
클러스터에서 키가 슬롯에 할당되는 것과 동일한 방식으로 채널이 할당되며, 같은 슬롯을 가지고 있는 노드 간에만 `pub/sub` 메시지를 전파한다.  

![sharded_pub_sub.jpeg](../res/sharded_pub_sub.jpeg)  

```shell
# 발행자 클라이언트
127.0.0.1:6379> SPUBLISH apple a
0
```
```shell
# 수신자 클라이언트
127.0.0.1:6379> SSUBSCRIBE apple
ssubscribe
apple
1
```
```shell
# 발행자 클라이언트
127.0.0.1:6379> SPUBLISH apple a
1
```
```shell
smessage
apple
a
```

<br>

apple 채널은 apple 키 값을 할당 받을 수 있는 슬롯을 포함한 마스터 노드에 연결될 수 있도록 리다이렉트된다.  
`sharded pub/sub`를 사용한다면 클러스터 구조에서 `pub/sub` 되는 메시지는 모든 노드로 전파되지 않기 때문에 불필요한 복제를 줄여 자원을 절약할 수 있다는 장점이 있다.  

<br>

## 💡 레디스의 `list`를 메시징 큐로 사용하기
### ✅ `list`의 `EX` 기능
> 이 내용은 트위터의 로직 관리자였던 라피 크리코리안(Raffic Krikorian)이 2013년에 QCon 웨비나에서 발표한 내용을 바탕으로 한다.

트위터는 각 유저의 타임라인 캐시 데이터를 레디스에서 `list` 자료구조로 관리한다.   
![twiter_timeline_cache.jpeg](../res/twiter_timeline_cache.jpeg)  
A를 팔로우하는 유저 B와 C의 타임라인 캐시 `list`에 A가 쓴 트윗의 데이터가 새로운 아이템으로 추가된다.  

**`RPUSHX`**  
각 타임라인 캐시에 데이터를 저장할 때 `RPUSH` 캐시가 아닌 `RPUSHX` 커맨드를 사용함을 주목해보자.  
`RPUSHX`는 데이터를 저장하고자 하는 `list`가 이미 존재할 때에만 아이템을 추가하는 커맨드다.  
자주 트위터를 들어오지 않는 D 유저에 대해서는 타임라인 캐시 데이터를 굳이 관리할 필요가 없기 때문에 캐시된 타임라인에만 데이터를 추가할 수 있다.  
```shell
# userB 타임라인 캐시 존재
127.0.0.1:6379> RPUSH Timelinecache:userB data1
1
# userC 타임라인 캐시 존재
127.0.0.1:6379> RPUSH Timelinecache:userC data2
1

# RPUSHX로 캐시에 아이템 추가 시작
127.0.0.1:6379> RPUSHX Timelinecache:userB data3
2
127.0.0.1:6379> RPUSHX Timelinecache:userC data3
2
# userD는 캐시 키가 존재하지 않아 캐시 아이템이 추가 안 됨
127.0.0.1:6379> RPUSHX Timelinecache:userD data3
0
```
➡ 사용자의 캐시가 이미 존재하는지의 유무를 애플리케이션에서 확인하는 과정 없이, 모든 로직을 레디스에서 제어할 수 있기 때문에 불필요한 확인 과정을 줄여 성능을 향상시킬 수 있게 된다.  

<br>

### ✅ `list`의 블로킹 기능
레디스를 이벤트 큐로 사용할 경우 블로킹 기능 또한 유용하게 사용할 수 있다.  
 
![event_queue_process.jpeg](../res/event_queue_process.jpeg)  

1. 이벤트 루프는 이벤트 큐에 새 이벤트가 있는지(큐가 비었는지) 체크하며, 새로운 이벤트가 없을 경우 정해진 시간(polling interval) 동안 대기한 뒤 다시 이벤트 큐에 데이터가 있는지 확인하는 과정을 반복한다.
2. 폴링 프로세스가 진행되는 동안 애플리케이션과 큐의 리소스가 불필요하게 소모될 수 있다.
3. 이벤트 큐에 이벤트가 들어와도, 폴링 이벤트 시간 동안은 대기한 뒤 다시 확인하는 과정을 거치기 때문에 이벤트를 즉시 처리할 수 없다는 단점이 있다.

**`BRPOP`, `BLPOP`**  
- 각각 `RPOP`, `LPOP`에 블로킹을 추가한 커맨드다.  
- 데이터를 요청했을 때 `list`에 데이터가 있으면 즉시 반환한다.
- 없을 경우
  - `list`에 데이터가 들어올 때까지 기다린 후에 들어온 값을 반환하거나,
  - 클라이언트가 설정한 타임아웃 시간만큼 대기한 후 `nil` 값을 반환한다.

```shell
# 수신자 클라이언트
# 데이터가 입력될 때까지 최대 5초 동안 대기하고, 5초가 경과되면 nil을 반환한다.
127.0.0.1:6379> BRPOP queue:a 5
```

```shell
# 발행자 클라이언트
127.0.0.1:6379> RPUSH queue:a hello_data
1
```

```shell
# 수신자 클라이언트
queue:a
hello_data
```

- 타임아웃 값을 0으로 설정하면 데이터가 리스트에 들어올 때까지 제한 없이 기다리라는 의미로 쓰인다.
- 하나의 리스트에 대해 여러 클라이언트가 동시에 블로킹될 수 있다. ➡ 리스트에 데이터가 입력되면 가장 먼저 요청을 보낸 클라이언트가 데이터를 가져간다.
- `BRPOP` 결과 2개의 데이터를 반환한다.
  1. `POP`된 리스트의 키 값
  2. 반환된 데이터의 값  
    동시에 여러 개의 리스트에서 대기할 수 있게 한다.

```shell
# 수신자 클라이언트
127.0.0.1:6379> BRPOP queue:a queue:b queue:c timeout 1000
```
```shell
# 발행자 클라이언트
127.0.0.1:6379> RPUSH queue:b EVENT_DATA
1
```
```shell
# 수신자 클라이언트
queue:b
EVENT_DATA
```

<br>

### ✅ `list`를 이용한 원형 큐
특정 아이템을 계속해서 반복 접근해야 하는 클라이언트, 혹은 여러 개의 클라이언트가 병렬적으로 같은 아이템에 접근해야 하는 클라이언트에서는 원형 큐(circular queue)를 이용해 아이템을 처리하고 싶을 수 있다.  

**`RPOPLPUSH`**  
```shell
127.0.0.1:6379> LPUSH clist A
1
127.0.0.1:6379> LPUSH clist B
2
127.0.0.1:6379> LPUSH clist C
3
127.0.0.1:6379> LRANGE clist 0 -1
C
B
A

127.0.0.1:6379> RPOPLPUSH clist clist
A
127.0.0.1:6379> LRANGE clist 0 -1
A
C
B
```

<br>

## 💡 Stream
### ✅ 레디스의 `stream`과 아파치 카프카
`Stream`은 레디스 5.0에서 새로 추가된 자료구조로, 대용량・대규모의 메시징 데이터를 빠르게 처리할 수 있도록 설계됐다.  
일반적으로 로그가 파일의 내용을 업데이트하거나 지우지 않고 쌓기만 하는 것처럼, `stream` 또한 데이터를 계속해서 추가하는 방식으로 저장되는 append-only 자료구조다.  

`stream`의 사용 목적에 따른 두 가지 활용 방식
1. 백엔드 개발자들이 대량의 데이터를 효율적으로 처리하는 플랫폼으로 활용할 수 있다.
2. 데이터 엔지니어들이 여러 생산자가 생성한 데이터를 다양한 소비자가 처리할 수 있게 지원하는 데이터 저장소 및 중간 큐잉 시스템으로 사용할 수 있다.

<br>

### ✅ 스트림이란?
> 컴퓨터 과학에서 스트림이란 연속적인 데이터의 흐름, 일정한 데이터 조각의 연속을 의미한다.

1. 텍스트 파일을 처리하는 애플리케이션
   - 파일 하나는 유한하지만 이를 읽어올 때 애플리케이션은 단어 단위, 또는 줄 단위로 데이터를 잘게 쪼개서 처리하기 때문에 프로그램은 바이트 스트림을 처리하는 것이라고 생각할 수 있다.
   - 끝이 정해지지 않고 계속해서 불규칙한 데이터를 연속으로 반복 처리할 때 이 또한 스트림 처리를 한다고 볼 수 있다.
2. 채팅 프로그램에서 JSON 파일을 스트리밍하는 과정
   - 채팅 앱에서 사용자는 아무때나 채팅을 보낼 수 있다.
   - 메신저 서버는 사람들이 계속 채팅하는 동안 끝없이 데이터를 처리할 수 있어야 한다.
   - JSON 스트림을 처리하는 것이라고 볼 수 있다.
3. 애플리케이션 내부에서 서버 간 데이터의 이동 - 웹 서버에서 받아온 결제 데이터
   - 분석 서버로 전달해야 할 수 있다.
   - 사용자의 결제 데이터를 이메일 서버로 넘겨야할 수 있다.
   - 서비스 간 데이터 전달 과정 또한 연속적인 데이터의 전달을 의미한다.
4. 예매 서비스에서의 스트림 데이터 처리
   - 웹사이트에서 유저가 트리거한 이벤트가 발생할 수 있다.
   - 신규 상품이 생성되거나 삭제되는 등 상품에 대한 이벤트가 생성될 수 있다.
   - 사용자가 결제 요청하면 결제를 처리하기 위한 이벤트도 생성된다.
     - 결제 프로세스 뿐만 아니라 웹 서버 프로세스에서도 처리돼야 한다.

<br>

### ✅ 데이터의 저장
**메시지의 저장과 식별**  
![kafka_redis_stream_data_recog.jpeg](../res/kafka_redis_stream_data_recog.jpeg)  
- 카프카(참고 링크: [카프카](https://github.com/kyeoungchan/note/tree/main/kafka))
  - 스트림 데이터는 토픽이라는 개념에 저장된다.
    - 토픽은 각각의 분리된 스트림을 뜻한다.
    - 같은 데이터를 관리하는 하나의 그룹이다.
  - 각 메시지는 0부터 시작해 증가하는 시퀀스 넘버로 식별할 수 있다.
    - 토픽 내의 파티션 안에서만 유니크하게 증가한다.  
      ➡ 토픽이 2개 이상의 파티션을 갖는다면 각 메시지는 하나의 토픽 내에서 유니크하게 식별되지 않는다.
- 레디스의 stream
  - string, list, has, set, sorted set 등과 마찬가지로 하나의 키에 연결된 하나의 자료구조다.
  - 각 메시지는 시간과 관련된 유니크한 ID를 가지며, 이 값은 중복되지 않는다.
    - `<millisecondsTime>-<sequenceNumber>`
      - `millisecondsTime` 파트: 실제 stream에 아이템이 저장될 시점의 레디스 로컬 시간이다.
      - `sequenceNumber` 파트: 동일한 `millisecondsTime` 시간에 여러 아이템이 저장될 수 있으므로, 같은 `millisecondsTime`에 저장된 데이터의 순서를 의미한다.  
        ➡ `sequenceNumber`는 64bit로, 사실상 하나의 `millisecondsTime` 내에 생성할 수 있는 항목 수에 제한이 없는 것과 같다.
    - ID 값이 곧 시간을 의미하기 때문에 시간을 이용해 특정 데이터를 검색할 수 있다.

<br>

### ✅ 스트림 생성과 데이터 입력
**카프카**  
생성자는 데이터를 토픽에 푸시한다.  
소비자는 토픽에서 데이터를 읽어간다.  
데이터를 저장하기 위해 토픽을 먼저 생성한 뒤, 프로듀서를 통해 메시지를 보낼 수 있다.
```shell
# 토픽 생성
$ kafka-topics --zookeeper 127.0.0.1:6000 --topic Email --create partitions 1 --replication-factor 1

# 데이터 추가
$ kafka-console-consumer --brokers-list 127.0.0.1:7000 --topic Email
> "I am first email"
> "I am second email"
```

<br>

**레디스 stream**  
레디스에서는 따로 stream을 생성하는 과정은 필요하지 않고, `XADD` 커맨드를 이용해 새로운 이름의 `stream`에 데이터를 저장하면 된다. (참고: [레디스에서 키를 관리하는 법](https://github.com/kyeoungchan/note/tree/main/redis/basic/key-management)의 "키의 자동 생성과 삭제")  
```shell
127.0.0.1:6379> XADD Email * subject "first" body "hello?"
1763856669820-0
```
- Email이라는 이름의 stream 생성과 동시에 메시지 추가
  - 같은 이름의 키가 존재했다면 이 커맨드는 기존 stream에 새로운 메시지를 추가한다.
- `*` 필드: 저장되는 데이터의 ID를 의미한다.
  - 값이 `*`라는 것은 레디스에서 자동 생성되는 타임스탬프 ID를 사용하겠다는 것을 의미한다.
  - 저장되는 데이터의 ID가 반환된다.
```shell
127.0.0.1:6379> XADD Push * userid 1000 ttl 3 body Hey
1763856688039-0
# 이미 존재하는 같은 Email Stream에 저장된다.
127.0.0.1:6379> XADD Email * subject "second" body "hi?"
1763856704137-0
```

![redis_stream_data_process.jpeg](../res/redis_stream_data_process.jpeg)  
앞선 커맨드로 레디스 stream에 데이터는 위와 같은 형식으로 저장됐다.

<br>

```shell
127.0.0.1:6379> XADD mystream 0-1 "hello" "world"
0-1
127.0.0.1:6379> XADD mystream 0-2 "hi" "redis"
0-2
```
자동으로 생성되는 ID가 아니라 직접 ID 값을 지정하고 싶다면 위와 같이 하면 된다.  
이 경우 지정할 수 있는 최소 ID 값은 0-1이며, 이후에 저장되는 stream의 ID는 이전에 저장됐던 값보다 작은 값으로 지정할 수 없다.

<br>

### ✅ 데이터의 조회
- 카프카
  - 소비자는 특정 토픽을 실시간으로 리스닝하며, 새롭게 토픽에 저장되는 메시지를 전달받을 수 있다.
  - 기본적으로는 리스닝을 시작한 시점부터 토픽에 새로 저장되는 메시지를 반환받도록 동작한다.
  - `--from-begining` 옵션을 이용하면 카프카에 저장돼 있는 모든 데이터를 처음부터 읽겠다는 것을 뜻한다.
  - 소비자는 더 이상 토픽에서 읽어올 데이터가 없으면 새로운 이벤트가 토픽에 들어올 때까지 계속 토픽을 리스닝하면서 기다린다.
```shell
$ kafka-console-producer --bootstrap-server 127.0.0.1:7000 --topic Email --from-begining
> "I am first email"
> "I am second email"
```

레디스 `stream`에서는 데이터를 읽어오는 방식이 두 가지가 있다.
1. 카프카에서처럼 실시간으로 처리되는 데이터를 리스닝 하는 방법
2. ID를 이용해 필요한 데이터를 검색하는 방법

<br>

**1. 실시간 리스닝**  
```shell
127.0.0.1:6379> XREAD [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] id [id ...]
```
`XREAD` 커맨드를 이용하면 실시간으로 stream에 저장되는 데이터를 읽어올 수 있다.
```shell
127.0.0.1:6379> XREAD BLOCK 0 STREAMS Email 0
Email
1763856669820-0
subject
first
body
hello?
1763856704137-0
subject
second
body
hi?
```
- `BLOCK 0`: 더 이상 stream에서 가져올 데이터가 없더라도 연결을 끊지 말고 계속 stream을 리스닝하라는 뜻이다.
  - 만약 `BLOCK 1000`을 입력했다면 들어오는 데이터가 없더라도 최대 1000ms(1초) 동안 연결을 유지하며 대기하라는 뜻이다.
- `Email 0`: Email이라는 stream에 저장된 데이터 중 ID가 0보다 큰 값을 읽어오라는 의미로, 처음부터 저장된 모든 데이터를 읽어오라는 것을 의미한다.
  - 만약 실행한 이후의 메시지만을 가져오고 싶다면 `0` 대신 특수 ID인 `$`를 입력하면 된다.  
    ➡ `$`: stream에 저장된 최대 ID를 의미한다.

```shell
# 1763856704136 보다 큰 데이터 ID를 갖는 데이터를 Email Stream에서 가져온다.
127.0.0.1:6379> XREAD BLOCK 0 STREAMS Email 1763856704136
Email
1763856704137-0
subject
second
body
hi?
```

<br>

**2. 특정한 데이터 조회**  
```shell
XRANGE key start end [COUNT count]
XREVRANGE key end start [COUNT count]
```
`XRANGE` 커맨드를 이용하면 ID를 이용해 원하는 시간대의 데이터를 조회할 수 있다.  
- `-`: 가장 작은 ID 값을 지정하고 싶을 때
- `+`: 제일 마지막 ID 값을 지정하고 싶을 때

`XREVRANGE`: `XRANGE`의 역순으로 데이터를 조회하고 싶을 때 사용한다.

<br>

```shell
127.0.0.1:6379> XRANGE Email - +
1763856669820-0
subject
first
body
hello?
1763856704137-0
subject
second
body
hi?
```

<br>

> `XRANGE Email - +` vs. `XREAD BLOCK 0 STREAMS Email 0`  
> `XREAD`를 사용했을 경우에는 반환할 데이터가 없으면 계속 기다리고 있지만,  
> `XRANGE`를 사용하면 커맨드를 수행하는 시점의 데이터가 없더라도 바로 종료된다.  
> ```shell
> 127.0.0.1:6379> XRANGE Email 1763935668105 +
> 
> 127.0.0.1:6379>
> ```

<br>

만약 입력한 데이터를 포함하지 않고, 그 다음 데이터부터 조회하고 싶을 때는 타임스탬프 값에 `(` 문자를 사용할 수 있다.  
```shell
# 1763935668104 포함한 데이터 반환
127.0.0.1:6379> XRANGE Email 1763935668104 +
1763935668104-0
subject
third
body
really?

# 1763935668104 이후의 데이터 반환
127.0.0.1:6379> XRANGE Email (1763935668104 +

# id에 시퀀스 넘버까지 전부 붙여도 결과는 같다.
127.0.0.1:6379> XRANGE Email (1763935668104-0 +

127.0.0.1:6379> XRANGE Email 1763935668104-0 +
1763935668104-0
subject
third
body
really?
```

<br>

### ✅ 소비자와 소비자 그룹

팬아웃(fan-out): 같은 데이터를 여러 소비자에게 전달하는 것  
![kafka_fan_out.jpeg](../res/kafka_fan_out.jpeg)  
카프카에서 여러 소비자에게 팬아웃을 하는 방식은 위 그림과 같다.  

<br>

![redis_stream_fan_out.jpeg](../res/redis_stream_fan_out.jpeg)  
레디스에서도 마찬가지의 방식으로 팬아웃이 가능하다.

<br>

이벤트의 처리 성능을 높이기 위해 여러 소비자가 여러 이벤트를 병렬적으로 처리되도록 구성할 수 있다.  
레디스 stream은 데이터가 저장될 때마다 고유한 ID를 부여받아 순서대로 저장되고, 소비자에게 데이터를 전달할 때도 그 순서는 항상 보장된다.  
하지만 카프카에서 유니크 키는 파티션 내에서만 보장되기 때문에 소비자가 여러 파티션에서 토픽을 읽어갈 때에는 데이터의 순서를 보장할 수 없다.

![kafka_partition.jpeg](../res/kafka_partition.jpeg)  
- 메시지는 토픽에 저장될 때 해시함수에 의해 3개의 파티션에 랜덤하게 분배된다.
- 소비자가 토픽에서 데이터를 소비할 때에는 파티션의 존재를 알지 못한다.  
  ➡ 토픽내의 전체 파티션에서 데이터를 읽어온다.
- 여러 파티션 간 데이터의 정렬은 보장되지 않기 때문에 소비자가 데이터를 읽어올 때에는 정렬이 보장되지 않는 데이터를 읽어오게 된다.
- 카프카에서 메시지 순서가 보장되도록 데이터를 처리하기 위해서는 소비자 그룹을 사용해야 한다.

<br>

**Kafka 소비자 그룹**  

카프카에서는 소비자 그룹에 여러 소비자를 추가할 수 있으며, 이때 소비자는 토픽 내의 파티션과 일대일로 연결된다.
![kafka_consumer_group.jpeg](../res/kafka_consumer_group.jpeg)  
소비자 그룹을 통해 각 소비자가 파티션과 연결되면 순서가 보장된 메시지를 받을 수 있게 된다.  

> 처리 순서가 중요한 것들은 같은 파티션으로 보내는 것이 보장돼야 한다.  
> ➡ 파티셔닝 전략이 몇 가지 있다. (추후 카프카 다룰 때 작성 예정)  
> 그리고 각 파티션은 그룹 내 하나의 컨슈머에만 할당되지만, 컨슈머는 여러 파티션을 처리할 수 있다.

<br>

**레디스 Stream 소비자 그룹**  

레디스 stream에서도 소비자 그룹이라는 개념이 존재하지만, kafka와는 달리 메시지가 전달되는 순서를 신경쓰지 않아도 된다.  
➡ 소비자 그룹 내의 한 소비자는 다른 소비자가 아직 읽지 않은 데이터만을 읽어간다.

![redis_stream_consumer_group.jpeg](../res/redis_stream_consumer_group.jpeg)  

```shell
127.0.0.1:6379> XGROUP CREATE Email EmailServiceGroup $
OK
```
- `XGROUP`: `Email` stream을 읽어가는 `EmailServiceGroup`이라는 소비자 그룹을 생성
- `$`: 현재 시점 이후의 데이터부터 리스닝하겠다는 뜻

<br>

```shell
127.0.0.1:6379> XREADGROUP GROUP EmailServiceGroup emailService1 COUNT 1 STREAMS Email >
```
- `XREADGROUP`: `XREAD`와 같은 형태로 데이터를 응답하지만, 저장한 소비자 그룹을 통해서 데이터를 읽기를 원한다는 뜻이다.
  - `EmailServiceGroup`에 속한 `emailService1`이라는 이름의 소비자가 `Email` stream에 있는 1개의 메시지를 읽고자 한다는 뜻이다.
  - 매번 소비자가 소비자 그룹을 이용해 작업을 수행할 때마다 그룹 내에서 이 소비자를 고유하게 식별할 수 있는 이름을 지정해야 한다.
  - 만약 다른 소비자에게 읽히지 않은 데이터가 있다면 데이터를 1개 가져오고, 없다면 `nil` 값을 반환한다.
- `COUNT`: `Kafka`와는 달리 각 소비자는 `COUNT` 커맨드를 통해 소비할 메시지의 개수를 지정할 수 있다.
- `STREAMS Email >`: `Email`이라는 이름의 stream에서 다른 소비자에게 전달되지 않았던 새로운 메시지를 전달하라는 의미다.
  - 소비자 그룹을 사용하는 이유가 다른 소비자에게 전달되지 않은 데이터를 가지고 오는 것  
    ➡ 대부분의 상황에서 `>`를 쓰게 될 것이다.
  - `>`외에 `0` 또는 다른 숫자 ID를 입력할 경우, 입력한 ID보다 큰 ID 중 대기 리스트(pending list)에 속하던 메시지를 반환한다.

<br>

위의 예제를 반환시켜보자.
```shell
# Email Stream에 4번째 메시지 저장
127.0.0.1:6379> XADD Email * subject "fourth" body "안녕?"
1763985441998-0

# emailService1을 통해 수신 결과 반환
127.0.0.1:6379> XREADGROUP GROUP EmailServiceGroup emailService1 COUNT 1 STREAMS Email >
Email
1763985441998-0
subject
fourth
body
안녕?

# emailService1을 통해 수신 결과 없음
127.0.0.1:6379> XREADGROUP GROUP EmailServiceGroup emailService1 COUNT 1 STREAMS Email >

```

<br>

- 소비자는 처음 언급될 때 자동으로 생성된다.
- `XREADGROUP`을 사용하면 읽어오는 동작 자체가 일종의 쓰기 커맨드다.  
  ➡ 이 커맨드는 마스터에서만 호출할 수 있다.

<br>

**부하 분산의 관점에서..**  
카프카: 파티션이라는 개념을 이용해 소비자의 부하 분산을 관리한다.  
레디스의 stream: 소비자 그룹이라는 개념을 이용해 여러 소비자에게 stream의 데이터를 분산시킬 수 있다.

- `Email`이라는 stream 메시지를 읽어가기 위한 소비자 그룹은 다수 각각 독립적으로 존재할 수 있다.
- 하나의 그룹에서 a라는 메시지를 읽었다면, 같은 그룹에서는 그 메시지를 다시 읽을 수 없지만, 그 그룹에 속하지 않는 소비자는 해당 메시지를 읽을 수 있다.
- 하나의 소비자 그룹에서 여러 개의 stream을 리스닝하는 것도 가능하다.

<br>

```shell
# BIGroup 생성 및 stream 처음부터 읽기 설정
127.0.0.1:6379> XGROUP CREATE Email BIGroup 0
OK
127.0.0.1:6379> XGROUP CREATE Push BIGroup 0
OK

# Email과 Push 2개의 stream 리스닝
127.0.0.1:6379> XREADGROUP GROUP BIGroup BT1 COUNT 2 STREAMS Email Push > >
Email
1763856669820-0
subject
first
body
hello?
1763856704137-0
subject
second
body
hi?
Push
1763856688039-0
userid
1000
ttl
3
body
Hey
```

<br>

### ✅ ACK와 보류 리스트
> 여러 서비스가 메시지 브로커를 이용해 데이터를 처리할 때, 예상치 못한 장애로 인해 시스템이 종료됐을 경우, 이를 인지하고 재처리할 수 있는 기능이 필요하다.

메시지 브로커
- 각 소비자에게 어떤 메시지까지 전달됐는지 알고 있어야 한다.
- 전달된 메시지의 처리 유무를 인지하고 있어야 한다.

<br>

![pending_list_ack.jpeg](../res/pending_list_ack.jpeg)  

1. 보류 리스트(`pending list`): 각 소비자별로 읽어간 메시지에 대한 리스트를 새로 생성한다.
2. `last_delivered_id`: 마지막으로 읽어간 데이터의 ID  
   ➡ 동일한 메시지를 중복으로 전달하지 않게 한다.

<br>

![pending_list_ack2.jpeg](../res/pending_list_ack2.jpeg)  
만약 이메일 서비스2가 stream에게 데이터가 처리됐다는 뜻의 `ACK`를 보내면 레디스 stream은 이메일 서비스2의 보류 리스트에서 `ACK`를 받은 메시지를 삭제한다.

<br>

> 만약 이메일 서비스에 문제가 발생해 서비스를 재부팅해야 하는 상황에서, stream의 보류 리스트에 데이터가 남아있는 경우 해당 데이터를 먼저 불러와 처리하는 작업을 선행적으로 수행  
> ➡ 예상치 못한 서비스 중단에도 모든 메시지를 놓치지 않고 처리할 수 있게 된다.

> 만약 이메일 서버 1이 장애가 발생해 해당 서버를 당분간 사용하지 못하는 경우, 1번 서버가 작업중이던 메시지를 다른 서버에서 처리해야할 수 있다.  
> 1번 서버의 보류 리스트에 남아있는 메시지가 있는지 확인하는 과정을 거치면 된다.


<br>

```shell
# 소비자 그룹에서 보류 중인 리스트가 있는지 확인
127.0.0.1:6379> XPENDING Email EmailServiceGroup
1
1763985441998-0
1763985441998-0
emailService1
1
```
반환되는 값
1. 현재 소비자 그룹에서 `ACK`를 받지 못해 보류 중인 메시지의 개수
2. 보류 중인 메시지 ID의 최솟값
3. 보류 중인 메시지 ID의 최댓값
4. 각 소비자별로 보류 중인 리스트 개수(위의 경우 소비자가 하나뿐이라 그냥 1 반환)

<br>

```shell
# 메시지 처리
127.0.0.1:6379> XACK Email EmailServiceGroup 1763985441998-0
1

# 소비자 그룹에서 보류 중인 리스트가 있는지 확인 결과 없음
127.0.0.1:6379> XPENDING Email EmailServiceGroup
0



```
`XACK` 커맨드를 통해 데이터가 처리됐음을 알려줄 수 있다.

<br>

> Kafka도 레디스 Stream처럼 파티션별로 오프셋을 관리한다.  
> 카프카는 내부적으로 `__consumer_offsets`라는 토픽에 데이터를 기록한다.  
> 카프카에서 오프셋은 소비자가 마지막으로 읽은 위치가 아니라 다음으로 읽어야 할 위치를 기록한다는 점이 다르다.

<br>

> 레디스 stream에서의 `at most once` vs `at least once` vs `exactly once`  
> - `at most once`
>   - 메시지를 최대 한 번 보내는 것을 의미한다.
>   - 소비자는 메시지를 받자마자 실제 처리하기 전에 먼저 `ACK`를 보낸다.
>   - 소비자에 문제 생겨 메시지가 일부 손실되더라도 빠른 응답이 필요할 때 선택하는 전략이다.
> - `at least once`
>   - 소비자는 받은 메시지를 모두 처리한 뒤 `ACK`를 보낸다.
>   - 실제로 메시지가 처리됐지만 `ACK`를 전송하기 전에 소비자가 종료되는 상황  
>     ➡ 보류 리스트에 처리된 메시지도 남아 있어도 한 번 더 처리하게 되는 상황이 발생할 수 있다.
>   - 멱등성이 보장되는 서비스라면 상관없지만, 그렇지 않을 경우 문제가 생길 수 있다.
> - `exactly once`
>   - 모든 메시지가 무조건 한 번씩 전송되는 것을 보장한다.
>   - 레디스의 `set` 등의 추가 자료 구조를 이용해 이미 처리된 메시지인지 확인하는 과정이 필요할 수 있다.

<br>

### ✅ 메시지의 재할당
소비자에게 장애가 날 경우, 보류 리스트를 참고해서 다른 소비자가 해당 메시지를 처리해야할 때가 있다.  
이때 `XCLAIM` 커맨드를 사용한다.  

<br>

```shell
XCLAIM key group consumer min-idle-time id [id ...]
```
`min-idle-time`: 최소 대기 시간을 지정해야 한다.
- 메시지가 보류 상태로 최소 대기 시간을 초과한 경우에만 소유권을 변경한다.
- 같은 메시지가 2개의 다른 소비자에게 중복으로 할당되는 것을 방지한다.

<br>

```shell
127.0.0.1:6379> XCLAIM Email EmailServiceGroup EmailService3 360000 1763985441998-0
```

<br>

**메시지의 자동 재할당**  
`XAUTOCLAIM`: 소비자가 직접 보류했던 메시지 중 하나를 자동으로 가져와서 처리할 수 있도록 한다.  
보류중인 다음 메시지의 ID를 반환한다.
```shell
XAUTOCLAIM key group consumer min-idle-time start [COUNT count] [JUSTID]
```
최소 대기 시간을 만족하는 보류 중인 메시지가 있다면 지정한 소비자에 소유권을 재할당하는 방식으로 동작한다.  
➡ `XCLAIM`으로 할당하는 방식에 비해 대상 메시지 ID를 직접 입력해줄 필요가 없다.  

<br>

```shell
127.0.0.1:6379> XAUTOCLAIM Email EmailServiceGroup EmailService1 360000 0-0 count 1
0-0
```
1. 첫 번째 반환 값으로 대상 보류 메시지의 ID가 반환된다.  
   더이상 대기 중인 보류 메시지가 없을 경우 `0-0` 반환
2. 소유권이 이전된 메시지의 정보를 제공한다.


<br>

**메시지의 수동 재할당**  
stream 내의 각 메시지는 `counter`라는 값을 각각 가지고 있다.  
➡ `XREADGROUP`을 이용해 소비자에게 할당하거나, `XCLAIM`을 이용해 재할당할 경우 1씩 증가한다.

<br>

만약 메시지에 문제가 있어 처리되지 못할 경우 메시지는 여러 소비자에게 할당되기를 반복하면서 `counter` 값이 증가하게 된다.  
➡ `counter`가 특정 값에 도달하면 이 메시지를 특수한 다른 stream으로 보내, 관리자가 추후에 처리해야할 수도 있는데, 이런 메시지를 `dead letter` 라고 한다.

<br>

### ✅ stream 상태 확인
> 일반적인 메시징 시스템이 그렇듯, 어떤 소비자가 활성화됐는지, 보류된 메시지는 어떤 건지, 어떤 소비자 그룹이 메시지를 처리하고 있는지 등의 상태를 체크할 수 없다면 stream은 관리하기 굉장히 빡세질 것이다.

`XINFO`를 통해 stream의 여러 상태를 확인할 수 있다.
```shell

127.0.0.1:6379> XINFO HELP
XINFO <subcommand> [<arg> [value] [opt] ...]. Subcommands are:
CONSUMERS <key> <groupname>
    Show consumers of <groupname>.
GROUPS <key>
    Show the stream consumer groups.
STREAM <key> [FULL [COUNT <count>]
    Show information about the stream.
HELP
    Print this help.
```

<br>

```shell
127.0.0.1:6379> XINFO consumers email EmailServiceGroup
ERR no such key

# 소비자 그룹에 속한 소비자의 정보 확인
127.0.0.1:6379> XINFO consumers Email EmailServiceGroup
name
EmailService1
pending
0
idle
404464
inactive
-1

name
EmailService3
pending
0
idle
900010
inactive
-1

name
emailService1
pending
0
idle
4588118
inactive
4591400
```

<br>

```shell
# stream에 속한 전체 소비자 그룹 list
127.0.0.1:6379> XINFO GROUPS Email
name
BIGroup
consumers
1
pending
2
last-delivered-id
1763856704137-0
entries-read
2
lag
3

name
EmailServiceGroup
consumers
3
pending
0
last-delivered-id
1763985441998-0
entries-read
4
lag
1
```

<br>

```shell
# stream 자체의 정보
127.0.0.1:6379> XINFO STREAM Email
length
5
radix-tree-keys
1
radix-tree-nodes
2
last-generated-id
1763989625739-0
max-deleted-entry-id
0-0
entries-added
5
recorded-first-entry-id
1763856669820-0
groups
2
first-entry
1763856669820-0
subject
first
body
hello?
last-entry
1763989625739-0
subject
fifth
body
안녕2222녕?
```


<br>

**참고 자료**  
[개발자를 위한 레디스](https://product.kyobobook.co.kr/detail/S000210785682)  
메시지 큐 시스템 비교 및 Kafka 활용법