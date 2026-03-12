# 🧑🏻‍💻 트랜잭션
<hr>

- [✅ 트랜잭션의 주요 특성](#-트랜잭션의-주요-특성)
- [✅ MySQL에서의 트랜잭션](#-mysql에서의-트랜잭션)
- [✅ 주의사항](#-주의사항)

> [!NOTE]
> - 트랜잭션은 데이터베이스 시스템에서 **하나의 논리적 기능**을 정상적으로 수행하기 위한 **작업의 기본 단위**  
> - 트랜잭션은 작업의 **완전성**을 보장해주는 것이다.
> - 논리적인 작업 셋을 모두 완벽하게 처리하거나, 처리하지 못할 경우에는 원 상태로 복구해서 작업의 일부만 적용되는 현상(Partial update)이 발생하지 않게 만들어주는 기능이다.
> - 트랜잭션은 꼭 여러 개의 변경 작업을 수행하는 쿼리가 조합됐을 때만 의미있는 개념이 아니고, 논리적인 작업 셋 자체가 100% 적용되거나(COMMIT을 실행했을 때), 아무것도 적용되지 않아야(ROLLBACK 또는 ROLLBACK시키는 오류가 발생했을 때) 함을 보장해주는 것이다.

> [!TIP]
> 잠금(Lock)과 트랜잭션은 서로 비슷한 개념 같지만, 사실 잠금은 동시성을 제어하기 위한 기능이고, 트랜잭션은 데이터의 정합성을 보장하기 위한 기능이다.  
> 격리 수준이라는 것은 하나의 트랜잭션 내에서 또는 여러 트랜잭션 간의 작업 내용을 어떻게 공유하고 차단할 것인지를 결정하는 레벨이다.


## ✅ 트랜잭션의 주요 특성
<hr>

- 원자성(Atomicity)
  - 분해가 불가능한 작업의 최소단위
  - 연산 전체가 성공 또는 실패(All or Nothing)
  - 하나라도 실패할 경우 전체가 취소되어야 하는 특성
- 일관성(Consistency)
  - 트랜잭션이 실행 성공 후 항상 일관된 데이터베이스 상태를 보존해야하는 특성
  - `NOT NULL` 같은 데이터 타입, 제약 조건 등 규칙이 일관되게 유효해야 한다.
- 격리성(Isolation)
  - 트랜잭션 실행 중 생성하는 연산의 중간 결과를 다른 트랜잭션이 접근 불가한 특성
- 영속성(Durability)
  - 성공이 완료된 트랜잭션의 결과는 영속적으로 데이터베이스에 저장하는 특성

<br>

## ✅ MySQL에서의 트랜잭션
<hr>

```mysql
mysql> CREATE TABLE tab_myisam ( fdpk INT NOT NULL, PRIMARY KEY (fdpk) ) ENGINE=MyISAM;
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO tab_myisam (fdpk) VALUES (3);
Query OK, 1 row affected (0.00 sec)

mysql> CREATE TABLE tab_innodb ( fdpk INT NOT NULL, PRIMARY KEY (fdpk) ) ENGINE=INNODB;
Query OK, 0 rows affected (0.02 sec)

mysql> INSERT INTO tab_innodb (fdpk) VALUES (3);
Query OK, 1 row affected (0.01 sec)

-- // AUTO-COMMIT 활성화
mysql> SET autocommit=ON;
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO tab_myisam (fdpk) VALUES (1), (2), (3);
ERROR 1062 (23000): Duplicate entry '3' for key 'tab_myisam.PRIMARY'
mysql> INSERT INTO tab_innodb (fdpk) VALUES (1), (2), (3);
ERROR 1062 (23000): Duplicate entry '3' for key 'tab_innodb.PRIMARY'
mysql> SELECT * FROM tab_myisam;
+------+
| fdpk |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)

mysql> SELECT * FROM tab_innodb;
+------+
| fdpk |
+------+
|    3 |
+------+
1 row in set (0.00 sec)
```

> [!IMPORTANT]
> MyISAM 테이블에서 실행되는 쿼리는 이미 INSERT된 '1'과 '2'를 그대로 두고 쿼리 실행을 종료해버린다.  
> MEMORY 스토리지 엔진을 사용하는 테이블도 MyISAM 테이블과 동일하게 작동한다.  
> 하지만 InnoDB는 쿼리 중 일부라도 오류가 발생하면 전체를 원 상태로 만든다는 트랜잭션의 원칙대로 INSERT 문장을 실행하기 전 상태로 그대로 복구했다.  
> 트랜잭션이란 애플리케이션 개발에서 고민해야 할 문제를 줄여주는 아주 필수적인 DBMS의 기능이다.  

```java
INSERT INTO tab_a...;
IF(_is_insert1_succeed) {
    INSERT INTO tab_b...;
    IF(_is_insert2_succeed) {
        // 처리 완료
    }ELSE {
        DELETE FROM tab_a WHERE ...;
        IF(_is_delete_succeed){
            // 처리 실패 및 tab_a, tab_b 모두 원상 복구 완료
        }
        ELSE {
            // 해결 불가능한 심각한 상황 발생
            // tab_b에 INSERT는 안되고, 하지만 tab_a에는 INSERT 돼버렸는데 삭제는 안 되고...
        }
    }
}
```

> [!NOTE]
> 위 애플리케이션 코드처럼 트랜잭션에 지원되지 않는 MyISAM 레코드를 INSERT할 때 위와 같이 하지 않으면 방법이 없다.  
> 하지만 위의 코드를 트랜잭션이 지원되는 InnoDB 테이블에서 처리한다면 아래와 같이 간단한 코드로 완벽한 구현이 가능하다.

```java
try {
    START TRANSACTION;
    INSERT INTO tab_a...;
    INSERT INTO tab_b...;
    COMMIT;
} catch (exception) {
    ROLLBACK;    
}
```

<br>

## ✅ 주의사항
<hr>

> [!TIP]
> 트랜잭션 또한 DBMS의 커넥션과 동일하게 꼭 필요한 최소의 코드에만 적용하는 것이 좋다.  
> ➡ 프로그램 코드에서 트랜잭션의 범위를 최소하라는 의미다.


```text
1) 처리 시작
    ➡ 데이터베이스 커넥션 생성
    ➡ 트랜잭션 시작
2) 사용자의 로그인 여부 확인
3) 사용자의 글쓰기 내용의 오류 여부 확인
4) 첨부로 업로드된 파일 확인 및 저장
5) 사용자의 입력 내용을 DBMS에 저장
6) 첨부 파일 정보를 DBMS에 저장
7) 저장된 내용 또는 기타 정보를 DBMS에서 조회
8) 게시물 등록에 대한 알림 메일 발송
9) 알림 메일 발송 이력을 DBMS에 저장
    ⬅ 트랜잭션 종료(COMMIT) 
    ⬅ 데이터베이스 커넥션 반납 
10) 처리 완료
```

> [!NOTE]
> 위 처리 절차 중에서 DBMS 트랜잭션 처리에 좋지 않은 영향을 미치는 부분을 살펴보자.
> - 실제로 DBMS에 데이터를 저장하는 작업(트랜잭션)은 5번부터 시작된다.
>   - 2, 3, 4번의 절차가 아무리 빨리 처리된다고 하더라도 DBMS의 트랜잭션에 포함시킬 필요는 없다.
>   - 일반적으로 DB 커넥션은 개수가 제한적이어서 각 단위 프로그램이 커넥션을 소유하는 시간이 길어질수록 사용 가능한 여유 커넥션의 개수는 줄어들 것이다.
> - 더 큰 위험은 8번 작업이다.
>   - 메일 전송이나 FTP 파일 전송 작업 또는 네트워크를 통해 원격 서버와 통신하는 등과 같은 작업은 어떻게 해서든 DBMS의 트랜잭션 내에서 제거하는 것이 좋다.
>   - 프로그램이 실행되는 동안 메일 서버와 통신할 수 없는 상황이 발생한다면 웹 서버뿐 아니라 DBMS 서버까지 위험해지는 상황이 발생할 것이다.
> - 이 처리 절차에는 DBMS의 작업이 크게 4가지가 있다.
>   1. 사용자가 입력한 정보를 저장하는 5번과 6번 작업은 반드시 하나의 트랜잭션으로 묶어야 한다.
>   2. 7번 작업은 저장된 데이터의 단순 확인 및 조회이므로 트랜잭션에 포함할 필요는 없다.
>   3. 9번 작업은 조금 성격이 다르기 때문에 이전 트랜잭션과 함께 묶을 필요는 없어 보인다.
>   4. 7번 작업은 단순 조회라고 본다면 별도로 트랜잭션을 사용하지 않아도 무방해 보인다.

```text
1) 처리 시작
2) 사용자의 로그인 여부 확인
3) 사용자의 글쓰기 내용의 오류 여부 확인
4) 첨부로 업로드된 파일 확인 및 저장
    ➡ 데이터베이스 커넥션 생성(또는 커넥션 풀에서 가져오기)
    ➡ 트랜잭션 시작
5) 사용자의 입력 내용을 DBMS에 저장
6) 첨부 파일 정보를 DBMS에 저장
    ⬅ 트랜잭션 종료(COMMIT) 
7) 저장된 내용 또는 기타 정보를 DBMS에서 조회
8) 게시물 등록에 대한 알림 메일 발송
    ➡ 트랜잭션 시작
9) 알림 메일 발송 이력을 DBMS에 저장
    ⬅ 트랜잭션 종료(COMMIT) 
    ⬅ 데이터베이스 커넥션 반납 
10) 처리 완료
```




<br>

**출처**  
[Real MySQL 8.0](https://product.kyobobook.co.kr/detail/S000001766482)