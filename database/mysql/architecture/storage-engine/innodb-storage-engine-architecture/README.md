# 🧑🏻‍💻 InnoDB 스토리지 엔진 아키텍처
<hr>

- [✅ 프라이머리 키에 의한 클러스터링](#-프라이머리-키에-의한-클러스터링)
- [✅ 외래 키 지원](#-외래-키-지원)
- [✅ MVCC(Multi Version Concurrency Control)](#-mvccmulti-version-concurrency-control)
- [✅ 잠금 없는 일관된 읽기(Non-Locking Consistent READ)](#-잠금-없는-일관된-읽기non-locking-consistent-read)
- [✅ 자동 데드락 감지](#-자동-데드락-감지)
- [✅ InnoDB 버퍼 풀](https://github.com/kyeoungchan/note/tree/main/database/mysql/architecture/storage-engine/innodb-storage-engine-architecture/innodb-buffer-pool)

## ✅ 프라이머리 키에 의한 클러스터링
InnoDB의 모든 테이블은 기본적으로 프라이머리 키를 기준으로 클러스터링되어 저장된다.  
➡ 프라이머리 키 값 순서대로 디스크에 저장  
➡ 모든 세컨더리 인덱스는 레코드의 주소 대신 프라이머리 키의 값을 논리적인 주소로 사용

❗️ 프라이머리 키가 클러스터링 인덱스이기 때문에 프라이머리 키를 이용한 레인지 스캔은 상당히 빨리 처리될 수 있다.

<br>

## ✅ 외래 키 지원
외래 키에 대한 지원은 InnoDB 스토리지 엔진 레벨에서 지원하는 기능으로, MyISAM이나 MEMORY 테이블에서는 사용할 수 없다.  
InnoDB에서 외래 키는 부모 테이블과 자식 테이블 모두 해당 컬럼에 인덱스 생성이 필요하고, 변경 시에는 반드시 부모 테이블이나 자식 테이블에 데이터가 있는지 체크하는 작업을 진행한다.  
➡ 잠금이 여러 테이블로 전파되고, 그로 인해 데드락이 발생할 때가 많으므로 개발할 때도 외래 키의 존재에 주의하는 것이 좋다.

<br>

수동으로 데이터를 적재하거나 스키마 변경 등의 관리 작업이 실패할 수 있다.  
물론, 부모 테이블과 자식 테이블의 관계를 명확히 파악해서 순서대로 작업한다면 문제없이 실행할 수 있지만 외래 키가 복잡하게 얽힌 경우에는 그렇게 간단하지 않다.  
➡ 이런 경우에는 `foreign_key_checks` 시스템 변수를 OFF로 설정하면 외래 키 관계에 대한 체크 작업을 일시적으로 멈출 수 있다. 

```mysql
-- // 글로벌, 세션 모두 설정 가능한 변수이므로 현재 작업을 실행하는 세션에서만 외래 키 체크 기능을 비활성화시켜야 한다.
mysql> SET SESSION foreign_key_checks=OFF;
Query OK, 0 rows affected (0.00 sec)

mysql> SET SESSION foreign_key_checks=ON;
Query OK, 0 rows affected (0.00 sec)
```

❗️ 외래 키 체크를 일시적으로 중지한 상태에서 외래 키 관계를 가진 부모 테이블의 레코드를 삭제했다면 반드시 자식 테이블의 레코드도 삭제해서 일관성을 맞춘 후 다시 외래 키 체크 기능을 활성화해야 한다.  
❗️ `foreign_key_checks`가 비활성화되면 외래 키 관계의 부모 테이블에 대한 작업(`ON DELETE CASCADE`와 `ON UPDATE CASCADE` 옵션)도 무시하게 된다.

<br>

## ✅ MVCC(Multi Version Concurrency Control)
MVCC의 가장 큰 목적은 잠금을 사용하지 않는 일관된 읽기를 제공하는 데 있다.  
여기서 멀티 버전이라 함은 하나의 레코드에 대해 여러 개의 버전이 동시에 관리된다는 뜻이다.  
➡ InnoDB는 언두 로그(Undo log)를 이용해 이 기능을 구현한다.

<br>

```mysql
mysql> CREATE TABLE member (
    m_id   INT          NOT NULL,
    m_name VARCHAR(20)  NOT NULL,
    m_area VARCHAR(100) NOT NULL,
    PRIMARY KEY (m_id),
    INDEX ix_area (m_area)
);

mysql> INSERT INTO member (m_id, m_name, m_area)
       VALUES (12, '홍길동', '서울');

mysql> COMMIT;
```

<br>

Insert문이 실행되고 나면 아래와 같은 상황이 되어 있을 것이다.
![insert_into_member.png](../../../res/insert_into_member.png)

<br>

```mysql
mysql> UPDATE member SET m_area='경기' WHERE m_id=12;
```

<br>

![update.png](../../../res/update.png)

UPDATE 문장이 실행되면 커밋 실행 여부와 관계없이 InnoDB의 버퍼 풀은 새로운 값인 '경기'로 업데이트 된다.  
그리고 디스크의 데이터 파일에는 체크 포인트나 InnoDB의 Write 스레드에 의해 새로운 값으로 업데이트됐을 수도 있고 아닐 수도 있다(InnoDB가 ACID를 보장하기 때문에 일반적으로는 InnoDB의 버퍼 풀과 데이터 파일은 동일한 상태라고 가정해도 무방하다).

<br>

🤔 COMMIT이나 ROLLBACK이 되지 않은 상태에서 레코드를 조회하면 어디에 있는 데이터를 조회할까?  
```mysql
mysql> SELECT * FROM member WHERE m_id=12;
```
➡ MySQL 서버의 시스템 변수(`transaction_isolation`)에 설정된 [격리 수준(Isolation level)](https://github.com/kyeoungchan/note/tree/main/database/isolation-level)에 따라 다르다.  
- `READ_UNCOMMITTED`: InnoDB 버퍼 풀이 현재 가지고 있는 변경된 데이터를 읽어서 반환한다.
- `READ_COMMITTED`, `REPEATABLE_READ`, `SERIALIZABLE`: 아직 커밋되지 않았기 때문에 InnoDB 버퍼 풀이나 데이터 파일에 있는 내용 대신 변경되기 이전의 내용을 보관하고 있는 언두 영역의 데이터를 반환한다.

<br>

COMMIT 명령을 실행하면 InnoDB는 더이상의 변경 작업 없이 지금의 상태를 영구적인 데이터로 만들어버린다.  
➡ 커밋이 된다고 언두 영역의 데이터가 바로 삭제되는 것은 아니고, 이 언두 영역을 필요로 하는 트랜잭션이 더는 없을 때 비로소 삭제된다.
ROLLBACK을 실행하면 InnoDB의 언두 영역에 있는 백업된 데이터를 InnoDB 버퍼 풀로 다시 복구하고, COMMIT과 달리 언두 영역의 내용을 삭제해버린다.  


<br>

## ✅ 잠금 없는 일관된 읽기(Non-Locking Consistent READ)
InnoDB 스토리지 엔진은 MVCC 기술을 이용해 잠금을 걸지 않고 읽기 작업을 수행한다.  
특정 사용자가 레코드를 변경하고 아직 커밋을 수행하지 않았다 하더라도 이 변경 트랜잭션이 다른 사용자의 SELECT 작업을 방해하지 않는다.  
➡ 이것을 '잠금 없는 일관된 읽기'라고 표현하며, InnoDB에서는 변경되기 전의 데이터를 읽기 위해 언두 로그를 사용한다.  

❗️ 오랜 시간 동안 활성 상태인 트랜잭션으로 인해 MySQL 서버가 느려지거나 문제가 발생할 때가 가끔 있는데, 바로 일관된 읽기를 위해 언두 로그를 삭제하지 못하고 계속 유지해야 하기 때문에 발생하는 문제다.  
➡ 트랜잭션이 시작됐다면 가능한 한 빨리 롤백이나 커밋을 통해 트랜잭션을 완료하는 것이 좋다.

<br>

## ✅ 자동 데드락 감지
InnoDB 스토리지 엔진은 내부적으로 잠금이 교착 상태에 빠지지 않았는지 체크하기 위해 잠금 대기 목록을 그래프(Wait-for List) 형태로 관리한다.  
InnoDB 스토리지 엔진은 데드락 감지 스레드를 가지고 있어서 데드락 감지 스레드가 주기적으로 잠금 대기 그래프를 검사해 교착 상태에 빠진 트랜잭션들을 찾아서 그중 하나를 강제 종료한다.  
➡ 이때 어느 트랜잭션을 먼저 강제 종료할 것인지를 판단하는 기준은 트랜잭션의 언두 로그양이며, 언두 로그 레코드를 더 적게 가진 트랜잭션이 일반적으로 롤백의 대상이 된다.

> 참고로 InnoDB 스토리지 엔진은 상위 레이어인 MySQL 엔진에서 관리되는 테이블 잠금(LOCK TABLES 명령으로 잠긴 테이블)은 볼 수가 없어 데드락 감지가 불확실할 수 있는데, `innodb_table-locks` 시스템 변수를 활성화하면 테이블 레벨의 잠금까지 감지할 수 있다.  
> ➡ 특별한 이유가 없다면 `innodb_table-locks` 시스템 변수를 활성화시켜놓자.

<br>

동시 처리 스레드가 매우 많아지거나 각 트랜잭션이 가진 잠금의 개수가 많아지면 데드락 감지 스레드가 느려진다.  
➡ 데드락 감지 스레드는 잠금 목록을 검사해야 하기 때문에 잠금 상태가 변경되지 않도록 잠금 목록이 저장된 리스트(잠금 테이블)에 새로운 잠금을 걸고 데드락 스레드를 찾게 된다.  
➡ 데드락 감지 스레드가 느려지면 서비스 쿼리를 처리 중인 스레드는 더는 작업을 진행하지 못하고 대기하면서 서비스에 악영향을 미치게 된다.  

💡 이런 문제점을 해결하기 위해 MySQL 서버는 `innodb_deadlock_detect` 시스템 변수를 제공하며, OFF로 설정하면 데드락 감지 스레드는 더는 작동하지 않게 된다.  
➡ InnoDB 스토리지 엔진 내부에서 2개 이상의 트랜잭션이 상대방이 가진 잠금을 요구하는 상황인 데드락이 발생해도 누군가 중재해주지 않기 때문에 무한정 대기하게 될 것이다.  
➡ `innodb_lock_wait_timeout` 시스템 변수를 활성화하면 데드락 상황에서 일정 시간이 지나면 자동으로 요청이 실패하고 에러 메시지를 반환하게 된다.  
❗️ `innodb_deadlock_detect` 시스템 변수를 비활성화 한다면, `innodb_lock_wait_timeout` 시스템 변수의 기본값인 50초보다 훨씬 낮은 시간으로 변경해서 사용할 것을 권장한다.

<br>






<br>


**출처**  
[Real MySQL 8.0](https://product.kyobobook.co.kr/detail/S000001766482)
