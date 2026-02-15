# 🧑🏻‍💻 MySQL 시스템 변수
- [✅ 글로벌 변수와 세션 변수](#-글로벌-변수와-세션-변수)
> MySQL 서버는 기동하면서 설정 파일의 내용을 읽어 메모리나 작동 방식을 초기화하고, 접속된 사용자를 제어하기 위해 이러한 값을 별도로 저장해두는데, 이를 시스템 변수(System Variables)라고 한다.

```shell
# 시스템 변수 확인
mysql> SHOW GLOBAL VARIABLES;
...
```

|Name|Cmd-Line|Option File|System Var| Var Scope |Dynamic|
|---|---|---|---|-----------|---|
|active_all_roles_on_login|Yes|Yes|Yes| Global    |Yes|
|admin_address|Yes|Yes|Yes| Global    |No|
|admin_port|Yes|Yes|Yes| Global    |No|
|time_zone|||Yes| Both      |Yes|
|sql_login_bin|||Yes| Sessions  |Yes|
- `Cmd-Line`: MySQL 서버의 명령행 인자로 설정될 수 있는지 여부  
  ➡ `Yes`면 명령행 인자로 이 시스템 변수의 값을 변경하는 것이 가능하다.
- `option file`: `my.cnf`로 제어할 수 있는지 여부를 나타낸다.  
  ➡ 옵션 파일, 설정 파일, Configuration 파일 등은 모두 `my.cnf`를 지칭한다.
- `System Var`: 시스템 변수인지 아닌지를 나타낸다.
- `Var Scope`: 시스템 변수의 적용 범위를 나타낸다.
- `Dynamic`: 시스템 변수가 동적인지 정적인지를 구분한다.

<br>

## ✅ 글로벌 변수와 세션 변수
<hr>

- [💡 글로벌 범위의 시스템 변수](#-글로벌-변수와-세션-변수)
- [💡 세션 범위의 시스템 변수](#-세션-범위의-시스템-변수)
- [💡 `Both`로 명시된 시스템 변수](#-both로-명시된-시스템-변수)
### 💡 글로벌 범위의 시스템 변수
- 하나의 MySQL 서버 인스턴스에서 전체적으로 영향을 미치는 시스템 변수를 의미한다.
- 주로 MySQL 서버 자체에 관련된 설정일 때가 많다.
- 예시
  - `innodb_buffer_pool_size`: InnoDB 버퍼 풀 크기
    - InnoDB 버퍼 풀이란, 테이블이나 인덱스 데이터를 캐시 하는 메모리 영역을 뜻한다.
  - `key_buffer_size`: MyISAM의 키 캐시 크기

### 💡 세션 범위의 시스템 변수
- MySQL 클라이언트가 MySQL 서버에 접속할 때 기본으로 부여하는 옵션의 기본값을 제어하는 데 사용된다.
  - 클라이언트의 필요에 따라 개별 커넥션 단위로 다른 값을 변경할 수 있는 것이 세션 변수다.
- 한 번 연결된 커넥션의 세션 변수는 서버에서 강제로 변경할 수 없다.
- 예시
  - `autocommit`

### 💡 `Both`로 명시된 시스템 변수
- MySQL 서버가 기억만 하고 있다가 실제 클라이언트와의 커넥션이 생성되는 순간에 해당 커넥션의 기본값으로 사용되는 값이다.
  - 세션 범위의 시스템 변수는 MySQL 서버의 설정 파일에 초기값을 명시할 수 없으며, 커넥션이 만들어지는 순간부터 해당 커넥션에서만 유효한 설정 변수를 의미한다.

<br>

## ✅ 정적 변수와 동적 변수
MySQL 서버의 시스템 변수는 디스크에 저장돼 있는 설정 파일(my.cnf)을 변경하는 경우와, 이미 기동 중인 MySQL 서버의 메모리에 있는 MySQL 서버의 시스템 변수를 변경하는 경우로 구분할 수 있다.  
my.cnf에 작성된 내용을 변경하더라도 MySQL 서버가 재시작하기 전에는 적용되지 않는다.  
하지만, `SHOW` 명령으로 확인하거나, 동적 변수의 경우 `SET` 명령을 통해 값을 바꿀 수 있다.  
➡ 물론, `SET` 명령을 통해 변경되는 시스템 변수 값은 `my.cnf` 파일에 반영되는 것은 아니기 때문에 현재 기동 중인 MySQL 인스턴스 내에서만 유효하다.  
➡ MySQL 8.0부터는 `SET PERSIST` 명령을 이용하면 실행 중인 MySQL 서버의 시스템 변수를 변경함과 동싱 자동으로 설정 파일로도 기록된다.

```mysql
mysql> SHOW GLOBAL VARIABLES LIKE '%max_connections%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| max_connections        | 151   |
| mysqlx_max_connections | 100   |
+------------------------+-------+
2 rows in set (0.02 sec)

mysql> SET GLOBAL max_connections=500;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW GLOBAL VARIABLES LIKE 'max_connections';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 500   |
+-----------------+-------+
1 row in set (0.01 sec)
             
# GLOBAL 키워드를 빼면 세션 변수를 조회하거나 변경할 수 있다.
mysql> SHOW VARIABLES LIKE '%connection%';
+-----------------------------------+----------------------+
| Variable_name                     | Value                |
+-----------------------------------+----------------------+
| character_set_connection          | utf8mb4              |
| collation_connection              | utf8mb4_0900_ai_ci   |
| connection_memory_chunk_size      | 8192                 |
| connection_memory_limit           | 18446744073709551615 |
| global_connection_memory_limit    | 18446744073709551615 |
| global_connection_memory_tracking | OFF                  |
| max_connections                   | 500                  |
| max_user_connections              | 0                    |
| mysqlx_max_connections            | 100                  |
+-----------------------------------+----------------------+
9 rows in set (0.00 sec)
```

<br>

시스템 변수의 범위가 Both인 경우, 글로벌 시스템 변수의 값을 변경해도 이미 존재하는 커넥션 세션 변수 값은 변경되지 않고 그대로 유지된다.

```mysql
mysql> SHOW GLOBAL VARIABLES LIKE 'join_buffer_size';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| join_buffer_size | 262144 |
+------------------+--------+
1 row in set (0.01 sec)

mysql> SHOW VARIABLES LIKE 'join_buffer_size';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| join_buffer_size | 262144 |
+------------------+--------+
1 row in set (0.00 sec)

mysql> SET GLOBAL join_buffer_size=524288;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW GLOBAL VARIABLES LIKE 'join_buffer_size';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| join_buffer_size | 524288 |
+------------------+--------+
1 row in set (0.00 sec)

mysql> SHOW VARIABLES LIKE 'join_buffer_size';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| join_buffer_size | 262144 |
+------------------+--------+
1 row in set (0.00 sec)
```
`join_buffer_size`의 글로벌 변수 값은 변경됐지만, 현재 커넥션의 세션 변수 값은 그대로 유지하고 있다.

<br>



**출처**  
[Real MySQL 8.0](https://product.kyobobook.co.kr/detail/S000001766482)
