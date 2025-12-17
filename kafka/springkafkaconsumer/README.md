# 💻 스프링 카프카 컨슈머
스프링 카프카의 컨슈머는 기존 컨슈머를 2개의 타입으로 나누고 커밋을 7가지로 나누어 세분화했다.  

<br>

타입
1. 레코드 리스너(MessageListener)
   - 단 1개의 레코드를 처리한다.
   - 스프링 카프카 컨슈머의 기본 리스너 타입이다.
2. 배치 리스너(BatchMessageListener)
   - 기존 카프카 클라이언트 라이브러리의 `poll()` 메서드로 리턴받은 `ConsumerRecords`처럼 한 번에 여러 개 레코드들을 처리할 수 있다.


|   타입   | 리스너 이름                                         | 상세 설명                                                            |
|:------:|------------------------------------------------|------------------------------------------------------------------|
| RECORD | MessageListener                                | Record 인스턴스 단위로 프로세싱, auto commit 또는 컨슈머 컨테이너의 AckMode를 사용하는 경우  |
|        | AcknowledgingMessageListener                   | Record 인스턴스 단위로 프로세싱, manual commit을 하는 경우                       |
|        | ConsumerAwareMessageListener                   | Record 인스턴스 단위로 프로세싱, 컨슈머 객체를 활용하고 싶은 경우                         |
|        | AcknowledgingConsumerAwareMessageListener      | Record 인스턴스 단위로 프로세싱, manual commit을 사용하고 컨슈머 객체를 활용하고 싶은 경우     |
| BATCH  | BatchMessageListener                           | Records 인스턴스 단위로 프로세싱, auto commit 또는 컨슈머 컨테이너의 AckMode를 사용하는 경우 |
|        | BatchAcknowledgingMessageListener              | Records 인스턴스 단위로 프로세싱, manual commit을 사용하는 경우                    |
|        | BatchConsumerAwareMessageListener              | Records 인스턴스 단위로 프로세싱, 컨슈머 객체를 활용하고 싶은 경우                        |
|        | BatchAcknowledgingConsumerAwareMessageListener | Records 인스턴스 단위로 프로세싱, manual commit을 사용하고 컨슈머 객체를 활용하고 싶은 경우    |


<br>

스프링 카프카에선느 사용자가 사용할만한 커밋의 종료를 7가지로 세분화하고 미리 로직을 만들어놓았다.  
스프링 카프카에서는 커밋이라고 하지 않고 'AckMode'라고 부른다.  
**프로듀서에서 사용하는 옵션인 `acks` 옵션과 `AckMode`는 다르므로 혼동하지 않아야 한다.**  
스프링 카프카 컨슈머의 `AckMode` 기본값은 `BATCH`이고, 컨슈머의 `enable.auto.commit` 옵션은 `false`로 지정된다.

|     AckMode      | 설명                                                                                                                                                                                                                              |
|:----------------:|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|      RECORD      | 레코드 단위로 프로세싱 이후 커밋                                                                                                                                                                                                              |
|      BATCH       | `poll()` 메서드로 호출된 레코드가 모두 처리된 이후 커밋 <br> 스프링 카프카 컨슈머의 `AckMode` 기본값                                                                                                                                                             |
|       TIME       | 특정 시간 이후에 커밋 <br> 이 옵션을 사용할 경우에는 시간 간격을 선언하는 `AckTime` 옵션을 설정해야 한다.                                                                                                                                                             |
|      COUNT       | 특정 개수만큼 레코드가 처리된 이후에 커밋 <br> 이 옵션을 사용할 경우에 레코드 개수를 선언하는 `AckCount` 옵션을 설정해야 한다.                                                                                                                                                 |
|    COUNT_TIME    | TIME, COUNT 옵션 중 맞는 조건이 하나라도 나올 경우 커밋                                                                                                                                                                                           |
|      MANUAL      | `Acknowledgement.acknowledge()` 메서드가 호출되면 다음번 `poll()`때 커밋을 한다. <br> 매번 `acknowledge()` 메서드를 호출하면 BATCH 옵션과 동일하게 동작한다. <br> 이 옵션을 사용할 경우에는 `AcknowledgingMessageListener` 또는 `BatchAcknowledgingMessageListener`를 리스너로 사용해야 한다. |
| MANUAL_IMMEDIATE | `Acknowledgement.acknowledge()` 메서드를 호출한 즉시 커밋한다. <br> 이 옵션을 사용할 경우에는 `AcknowledgingMessageListener` 또는 `BatchAcknowledgingMessageListener`를 리스너로 사용해야 한다.                                                                      |

<br>

리스너를 생성하고 사용하는 방식에는 크게 2가지가 있다.
1. 기본 리스너 컨테이너를 사용하는 것
2. 컨테이너 팩토리를 사용하여 직접 리스너를 사용하는 것

[Spring Kafka Consumer 레포지토리](https://github.com/kyeoungchan/spring-kafka-consumer)를 참고하자.
