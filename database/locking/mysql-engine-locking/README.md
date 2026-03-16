# 🧑🏻‍💻 MySQL 엔진의 잠금
<hr>

- [✅ 글로벌 락](#-글로벌-락)
- [✅ 테이블 락](#-테이블-락)
- [✅ 네임드 락](#-네임드-락)
- [✅ 메타데이터 락](#-메타데이터-락)

> [!TIP]
> - 로킹 기법이란 트랜잭션이 사용하는 데이터 항목에 대하여 잠금(Lock)을 설정한 트랜잭션이 해제(Unlock)할 때까지 독점적으로 사용할 수 있게 상호배제 기능을 제공하는 기법이다.
> - [병쟁제어 기법](https://github.com/kyeoungchan/note/tree/main/database/concurrency-control) 중의 하나로 그 외에는 다음과 같이 있다.
>   - 낙관적 검증(Validation) 기법
>   - 타임 스탬프 순서(Timestamp ordering) 기법
>   - 다중버전 동시성 제어(MVCC) 기법

<br>

> [!NOTE]
> MySQL 엔진에서는 테이블 데이터 동기화를 위한 테이블 락 이외에도 테이블의 구조를 잠그는 메타데이터 락(Metadata Lock) 그리고 사용자의 필요에 맞게 사용할 수 있는 네임드 락(Named Lock)이라는 잠금 기능도 제공한다.

<br>

## ✅ 글로벌 락
<hr>

> [!IMPORTANT]
> 글로벌 락은 `FLUSH TABLES WITH READ LOCK` 명령으로 획득할 수 있으며, MySQL에서 제공하는 잠금 가운데 가장 범위가 크다.  
> 일단 한 세션에서 글로벌 락을 획득하면 다른 세션에서 `SELECT`를 제외한 대부분의 DDL 문장이나 DML 문장을 실행하는 경우 글로벌 락이 해제될 때까지 해당 문장이 대기 상태로 남는다.  
> 글로벌 락이 영향을 미치는 범위는 MySQL 서버 전체이며, 작업 대상 테이블이나 데이터베이스가 다르더라도 동일하게 영향을 미친다.  
> ➡️ 여러 데이터베이스에 존재하는 MyISAM이나 MEMORY 테이블에 대해 mysqldump로 일관된 백업을 받아야할 때는 글로벌 락을 사용해야 한다.

<br>

> [!CAUTION]
> `FLUSH TABLES WITH READ LOCK` 명령이 실행되기 전에 테이블이나 레코드에 쓰기 잠금을 거는 SQL이 실행됐다면 이 명령은 해당 테이블의 읽기 잠금을 걸기 위해 먼저 실행된 SQL과 그 트랜잭션이 완료될 때까지 기다려야 한다.  
> `FLUSH TABLES WITH READ LOCK` 명령은 테이블에 읽기 잠금을 걸기 전에 먼저 테이블을 플러시해야 하기 때문에 테이블에 실행 중인 모든 종류의 쿼리가 완료돼야 한다.  
> ➡️ 장시간 `SELECT` 쿼리가 실행되고 있을 때는 `SELECT` 쿼리가 종료될 때까지 기다려야 한다.

<br>

> [!WARN]
> 장시간 실행되는 쿼리와 `FLUSH TABLES WITH READ LOCK` 명령이 최악의 케이스로 실행되면 MySQL 서버의 모든 테이블에 대한 INSERT, UPDATE, DELETE 쿼리가 아주 오랜 시간 동안 실행되지 못하고 디라릴 수도 있다.  
> 글로벌 락은 MySQL 서버의 모든 테이블에 큰 영향을 미치기 때문에 웹 서비스용으로 사용되는 MySQL 서버에서는 가급적 사용하지 않는 것이 좋다.


<br>

> [!TIP]
> `mysqldump` 같은 백업 프로그램은 우리가 알지 못하는 사이에 `FLUSH TABLES WITH READ LOCK` 명령을 내부적으로 실행하고 백업할 때도 있다.  

<br>

> [!NOTE]
> InnoDB 스토리지 엔진은 트랜잭션을 지원하기 때문에 일관된 데이터 상태를 위해 모든 데이터 변경 작업을 멈출 필요는 없다.  
> 또한 MySQL 8.0부터는 InnoDB가 기본 스토리지 엔진으로 채택되면서 조금 더 가벼운 글로벌 락의 필요성이 생겼다.  
> ➡️ `Xtrabackup`이나 `Enterprise Backup`과 같은 백업 툴들의 안정적인 실행을 위해 백업 락이 도입됐다.

```mysql
mysql> LOCK INSTANCE FOR BACKUP;
Query OK, 0 rows affected (0.00 sec)

-- // 백업 실행

mysql> UNLOCK INSTANCE;
Query OK, 0 rows affected (0.00 sec)
```

> [!NOTE]
> 특정 세션에서 백업 락을 획득하면 모든 세션에서 다음과 같이 테이블의 스키마나 사용자의 인증 관련 정보를 변경할 수 없게 된다.  
> - 데이터베이스 및 테이블 등 모든 객체 생성 및 변경, 삭제
> - `REPAIR TABLE`과 `OPTIMIZE TABLE` 명령
> - 사용자 관리 및 비밀번호 변경
> 
> 하지만 백업 락은 일반적인 테이블의 데이터 변경은 허용된다.

<br>

> [!IMPORTANT]
> 일반적인 MySQL 서버의 구성은 소스 서버(Source Server)와 레플리카 서버(Replica Server)로 구성되는데, 주로 백업은 레플리카 서버에서 실행된다.  
> 하지만 백업이 `FLUSH TABLES WITH READ LOCK` 명령을 이용해 글로벌 락을 획득하면 복제는 백업 시간만큼 지연될 수밖에 없다.  
> 레플리카 서버에서 백업을 실행하는 도중에 소스 서버에 문제가 생기면 레플리카 서버의 데이터가 최신 상태가 될 때까지 서비스를 멈춰야할 수도 있다.  
> 갑자기 DDL 명령 하나로 인해 백업이 ㅅ리패하면 다시 그만큼 시간을 들여서 백업을 실행해야 한다.  
> ➡️ MySQL 서버의 백업 락은 이런 목적으로 도입됐으며, 정상적으로 복제는 실행되지만 백업의 실패를 막기 위해 DDL 명령이 실행되면 복제를 일시 중지하는 역할을 한다.

<br>

## ✅ 테이블 락
<hr>

> [!TIP]
> 테이블 락은 개별 테이블 단위로 설정되는 잠금이며, 명시적 또는 묵시적으로 특정 테이블의 락을 획득할 수 있다.  
> ➡️ 명시적 명령: `LOCK TABLES table_name [READ | WRITE]`  
> ➡️ 명시적으로 획득한 잠금은 `UNLOCK TABLES` 명령으로 잠금을 반납할 수 있다.  
> 명시적인 테이블 락도 특별한 상황이 아니면 애플리케이션에서 사용할 필요가 거의 없다.  
> 명시적으로 테이블을 잠그는 작업은 글로벌 락과 동일하게 온라인 작업에 상당한 영향을 미치기 때문이다.

<br>

> [!NOTE]
> 묵시적인 테이블 락은 MyISAM이나 MEMORY 테이블에 데이터를 변경하는 쿼리를 실행하면 발생한다.  
> 데이터를 변경한 후, 즉시 잠금을 해제하는 형태로 사용된다.  
> 
> 하지만 InnoDB 테이블의 경우 스토리지 엔진 차원에서 레코드 기반의 잠금을 제공하기 때문에 단순 데이터 변경 쿼리로 인해 묵시적인 테이블 락이 설정되지 않는다.  
> ➡️ 더 정확히는 InnoDB 테이블에도 테이블 락이 설정되지만 대부분의 데이터 변경(DML) 쿼리에서는 무시되고 스키마를 변경하는 쿼리(DDL)의 경우에만 영향을 미친다.

<br>

## ✅ 네임드 락
<hr>

> [!TIP]
> 네임드 락은 `GET_LOCK()` 함수를 이용해 임의의 문자열에 대해 잠금을 설정할 수 있다.  
> 이 잠금의 특징은 대상이 테이블이나 레코드 또는 `AUTO_INCREMENT`와 같은 데이터베이스 객체가 아니라는 것이다.  
> 네임드 락은 단순히 사용자가 지정한 문자열(String)에 대해 획득하고 반납하는 잠금이다.  
> 데이터베이스 서버 1대에 5대 웹 서버가 접속해서 서비스하는 상황에서 5대의 웹 서버가 어떤 정보를 동기화해야 하는 요건처럼 여러 클라이언트가 상도 동기화를 처리해야할 때 네임드 락을 이용하면 쉽게 해결할 수 있다.

<br>

## ✅ 메타데이터 락
<hr>

> [!TIP]
> 메타데이터 락은 데이터베이스 객체(테이블이나 뷰 등)의 이름이나 구조를 변경하는 경우에 획득하는 잠금이다.  
> 명시적으로 획득하거나 해제할 수 있는 것이 아니고, `RENAME TABLE table_a TO table_b`와 같이 테이블의 이름을 변경하는 경우 자동으로 획득하는 잠금이다.  
> `RENAME TABLE` 명령의 경우 원본 이름과 변경될 이름 두 개 모두 한꺼번에 잠금을 설정한다.

<br>

> [!IMPORTANT]
> 때로는 메타데이터 잠금과 InnoDB의 트랜잭션을 동시에 사용해야 하는 경우도 있다.  
> 예를 들어, 다음과 같은 구조의 INSERT만 실행되는 테이블을 가정해보자.  

```mysql
mysql> CREATE TABLE access_log (
    id BIGINT NOT NULL AUTO_INCREMENT,
    client_ip INT UNSIGNED,
    access_dttm TIMESTAMP,
    ...
    PRIMARY KEY (id)
   ); 
```

> [!NOTE]
> 이 테이블의 구조를 변경해야할 요건이 발생했다고 가정해보자.  
> 물론 MySQL 서버의 Online DDL을 이용해서 변경할 수도 있지만 시간이 너무 오래 걸리는 경우라면 언두 로그의 증가와 Online DDL이 실행되는 동안 누적된 Online DDL 버퍼의 크기 등 고민해야 할 문제가 많다.  
> 더 큰 문제는 MySQL 서버의 DDL은 단일 스레드로 작동하기 때문에 상당히 많은 시간이 소모될 것이라는 점이다.  
> ➡️ 이때는 새로운 구조의 테이블을 생성하고 먼저 최근(1시간 직전 또는 하루 전)의 데이터까지는 프라이머리 키인 id 값을 범위별로 나눠서 여러 개의 스레드로 빠르게 복사한다.

```mysql
-- // 테이블의 압축을 적용하기 위해 KEY_BLOCK_SIZE=4 옵션을 추가해 신규 테이블을 생성
mysql> CREATE TABLE access_log_new (
    id          BIGINT NOT NULL AUTO_INCREMENT,
    client_ip   INT UNSIGNED,
    access_dttm TIMESTAMP, 
    ...
    PRIMARY KEY(id)
    ) KEY_BLOCK_SIZE=4;

-- // 4개의 스레드를 이용해 id 범위별로 레코드를 신규 테이블로 복사
mysql_thread1> INSERT INTO access_log_new SELECT * FROM access_log WHERE id >= 0 AND id < 10000;
mysql_thread2> INSERT INTO access_log_new SELECT * FROM access_log WHERE id >= 10000 AND id < 20000;
mysql_thread3> INSERT INTO access_log_new SELECT * FROM access_log WHERE id >= 20000 AND id < 30000;
mysql_thread4> INSERT INTO access_log_new SELECT * FROM access_log WHERE id >= 30000 AND id < 40000;
```

> [!IMPORTANT]
> 그리고 나머지 데이터는 다음과 같이 트랜잭션과 테이블 잠금, `RENAME TABLE` 명령으로 응용 프로그램의 중단없이 실행할 수 있다.  
> 이때 "남은 데이터를 복사"하는 시간 동안은 테이블의 잠금으로 인해 INSERT를 할 수 없게 된다.  
> ➡️ 가능하면 미리 아주 최근 데이터까지 복사해둬야 잠금 시간을 최소화해서 서비스에 미치는 영향을 줄일 수 있다.

```mysql
-- // 트랜잭션을 autocommit으로 실행(BEGIN이나 START TRANSACTION으로 실행하면 안 됨)
mysql> SET autocommit = 0;

-- // 작업 대상 테이블 2개에 대해 테이블 쓰기  락을 획득
mysql> LOCK TABLES access_log WRITE, access_log_new WRITE;

-- // 남은 데이터들 복사
mysql> SELECT MAX(id) as @MAX_ID FROM access_log_new;
mysql> INSERT INTO access_log_new SELECT * FROM access_log WHERE id > @MAX_ID
mysql> COMMIT;

-- // 새로운 테이블로 데이터 복사가 완료되면 RENAME 명령으로 새로운 테이블을 서비스로 투입
mysql> RENAME TABLE access_log TO access_log_old, access_log_new to access_log;
mysql> UNLOCK TABLES;

-- // 불필요한 테이블 삭제
mysql> DROP TABLE access_log_old;
```



<br>

**출처**
- [Real MySQL 8.0](https://product.kyobobook.co.kr/detail/S000001766482)
