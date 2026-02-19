# 🧑🏻‍💻 스토리지 엔진
<hr>

스토리지 엔진이란, 데이터를 실제로 저장하고 관리하는 "엔진"이다.  

```mysql
CREATE TABLE users (
    id INT PRIMARY KEY
) ENGINE=InnoDB;  -- 생략해도 기본값이 InnoDB
```

<br>

MySQL은 두 부분으로 나뉜다.
1. MySQL 서버: SQL 쿼리 해석, 사용자 인증 등을 담당
2. 스토리지 엔진: 실제 데이터를 디스크에 쓰고 읽는 부분을 담당

<br>

Postgresql이나 오라클DB 등 다른 RDBMS와는 다르게, MySQL만 유일하게 플러그인 방식의 스토리지 엔진을 사용한다.
```shell
mysql> SHOW ENGINES;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| ndbcluster         | NO      | Clustered, fault-tolerant tables                               | NULL         | NULL | NULL       |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| ndbinfo            | NO      | MySQL Cluster system information storage engine                | NULL         | NULL | NULL       |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
```
> Support 컬럼
> - YES: MySQL 서버에 해당 스토리지 엔진이 포함돼 있고, 사용 가능으로 활성화된 상태임
> - DEFAULT: 'YES'와 동일한 상태이지만 필수 스토리지 엔진임을 의미함(즉, 이 스토리지 엔진이 없으면 MySQL이 시작되지 않을 수 있음을 의미한다)
> - NO: 현재 MySQL 서버에 포함되지 않음을 의미함
> - DISABLED: 현재 MySQL 서버에 포함됐지만 파라미터에 의해 비활성화된 상태임

<br>

## ✅ 컴포넌트
<hr>

MySQL 8.0부터 기존의 플러그인 아키텍처를 대체하기 위해 컴포넌트 아키텍처가 지원된다.  
➡ MySQL 플러그인의 단점은 다음과 같다.
- 플러그인은 오직 MySQL 서버와 인터페이스할 수 있고, 플러그인끼리는 통신할 수 없다.
- 플러그인은 MySQL 서버의 변수나 함수를 직접 호출하기 때문에 안전하지 않다.(캡슐화 안됨)
- 플러그인은 상호 의존 관계를 설정할 수 없어서 초기화가 어렵다.


```mysql
-- // validate_password 컴포넌트 설치
mysql> INSTALL COMPONENT 'file://component_validate_password';    

-- // 설치된 컴포넌트 확인
mysql> SELECT * FROM mysql.component;
+--------------+--------------------+------------------------------------+
| component_id | component_group_id | component_urn                      |
+--------------+--------------------+------------------------------------+
|            1 |                  1 | file://component_validate_password |
+--------------+--------------------+------------------------------------+
1 row in set (0.02 sec)
```






<br>


**출처**  
[Real MySQL 8.0](https://product.kyobobook.co.kr/detail/S000001766482)
