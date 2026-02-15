# 🧑🏻‍💻 스토리지 엔진
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