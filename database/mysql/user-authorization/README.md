# 🧑🏻‍💻 MySQL 사용자 및 권한
<hr>

- [✅ 사용자 식별](#-사용자-식별)
- [✅ 사용자 계정 관리](#-사용자-계정-관리)
- [✅ 비밀번호 관리](#-비밀번호-관리)
- [✅ 권한(Privilege)](#-권한privilege)
- [✅ 역할(Role)](#-역할role)

> MySQL에서는 다른 DBMS와는 달리 해당 사용자가 어느 IP에서 접속하고 있는지도 확인한다.  
> MySQL8.0부터는 권한을 묶어서 관리하는 역할(Role)의 개념이 도입되었다.

## ✅ 사용자 식별
<hr>

MySQL에서 계정을 언급할 때는 다음과 같이 항상 ID와 호스트를 함께 명시해야 한다.  
➡ ID와 IP 주소를 감싸는 역따옴표(`)는 MySQL에서 식별자를 감싸는 따옴표 역할을 하는데, 이는 종종 홑따옴표(')로도 바뀌어서 사용되기도 한다.  
만약 사용자 계정에 아래와 같은 계정만 등록돼 있다면 다른 컴퓨터에서는 svc_id라는 ID로 접속할 수 없음을 뜻한다.  
```
'svc_id'@'127.0.0.1'
```

<br>

만약 모든 외부 컴퓨터에서 접속이 가능한 사용자 게정을 생성하고 싶다면 사용자 계정의 호스트 부분을 `%` 문자로 대체하면 된다.  

### 🤔 사용자 계정 식별에 서로 동일한 ID가 있을 경우
예를 들어, 아래와 같이 2개의 사용자 계정이 있는 MySQL 서버가 있다.
```
'svc_id'@'192.168.0.10' (이 계정의 비밀번호는 123)
'svc_id'@'%' (이 계정의 비밀번호는 abc)
```

권한이나 계정 정보에 대해 MySQL은 범위가 가장 작은 것을 항상 먼저 선택한다.  
➡ 위의 경우, 'svc_id'@'192.168.0.10' 계정으로 인증하게 된다.  
➡ 중첩된 계정을 생성하지 않도록 주의해야 한다.

<br>

## ✅ 사용자 계정 관리
<hr>

- [💡 시스템 계정과 일반 계정](#-시스템-계정과-일반-계정)
- [💡 계정 생성](#-계정-생성)
### 💡 시스템 계정과 일반 계정
MySQL8.0부터 계정은 `SYSTEM_USER` 권한을 가지고 있느냐에 따라 시스템 계정(System Account)과 일반 계정(Regular Account)으로 구분된다.  
시스템 계정은 데이터베이스 서버 관리자를 위한 계정이며, 일반 계정은 응용 프로그램이나 개발자를 위한 계정 정도로 생각하면 이해하기 쉬울 것이다.  

<br>


#### 시스템 계정으로만 가능한 기능
- 계정 관리(계정 생성 및 삭제, 그리고 계정의 권한 부여 및 제거)
- 다른 세션(Connection) 또는 그 세션에서 실행 중인 쿼리를 강제 종료
- 스토어드 프로그램 생성 시 DEFINER를 타 사용자로 설정

<br>

#### MySQL 서버에 내장된 계정
'root'@'localhost'를 제외하고도 3개의 계정이 있다.
- 'mysql.sys'@'localhost': MySQL 8.0부터 기본으로 내장된 sys 스키마의 객체(뷰나 함수, 그리고 프로시저)들의 DEFINER로 사용되는 계정
- 'mysql.session'@'localhost': MySQL 플러그인이 서버로 접근할 때 사용되는 계정
- 'mysql.infoschema'@'localhost': `information_schema`에 정의된 뷰의 DEFINER로 사용되는 계정

위 3개의 계정은 처음부터 참겨있는 상태이기 때문에 의도적으로 풀지 않는 한 악의적인 용도로 사용할 수 없으므로 보안을 걱정하지 않아도 된다.
```mysql
mysql> SELECT user, host, account_locked FROM mysql.user WHERE user LIKE 'mysql.%';
+------------------+-----------+----------------+
| user             | host      | account_locked |
+------------------+-----------+----------------+
| mysql.infoschema | localhost | Y              |
| mysql.session    | localhost | Y              |
| mysql.sys        | localhost | Y              |
+------------------+-----------+----------------+
3 rows in set (0.01 sec)
```

<br>

### 💡 계정 생성
- 계정 생성: `CREATE USER`
- 권한 부여: `GRANT`

```mysql
-- 일반적으로 많이 사용되는 옵션을 가진 CREATE USER 명령
CREATE USER 'user'@'%'
    IDENTIFIED WITH 'mysql_native_passowrd' BY 'password'
    REQUIRE NONE
    PASSWORD EXPIRE INTERVAL 30 DAY
    ACCOUNT UNLOCK 
    PASSWORD HISTORY DEFAULT 
    PASSWORD REUSE INTERVAL DEFAULT 
    PASSWORD REQUIRE CURRENT DEFAULT;
```

<br>

#### `IDENTIFIED WITH`
사용자의 인증 방식과 비밀번호를 설정한다.  
`IDENTIFIED WITH` 뒤에는 반드시 인증 방식(인증 플러그인의 이름)을 명시해야 하는데, MySQL 서버의 기본 인증 방식을 사용하고자 한다면 `IDENTIFIED BY 'password'` 형식으로 명시해야 한다.  

- `Native Pluggable Authentication`: 단순히 비밀번호에 대한 해시(SHA-1 알고리즘) 값을 저장해두고, 클라이언트가 보낸 값과 해시값이 일치하는지 비교하는 인증 방식이다.
- `Caching SHA-2 Pluggable Authentication`: MySQL 8.0부터 조금 더 보완된 인증 방식으로, 암호화 해시값 생성을 위해 SHA-2(256비트) 알고리즘을 사용한다.
  - 저장된 해시값의 보안에 더 중점을 둔 알고리즘이다.
  - 내부적으로 Salt 키를 활용하며, 수천 번의 해시 계산을 수행해서 결과를 만들어내기 때문에 동일한 키 값에 대해서도 결과가 달라진다.  
    ➡ 해시값을 계산하는 방식이 시간 소모적이고 성능이 떨어져, 이를 보완하기 위해 해시 결과값을 메모리에 캐시해서 사용하게 된다.
  - 이 인증 방식을 사용하려면 SSL/TLS 또는 RSA 키페어를 반드시 사용해야하므로, 클라이언트에서 접속할 때 SSL 옵션을 활성화해야 한다.
- `PAM pluggable Authentication`: 유닉스나 리눅스 패스워드 또는 LDAP(Lightweight Directory Access Protocol) 같은 외부 인증을 사용할수 있게 해주는 인증 방식으로, MySQL 엔터프라이즈 에디션에서만 사용 가능하다.
- `LDAP Pluggable Authentication`: LDAP를 이용한 외부 인증을 사용할수 있게 해주는 인증 방식으로, MySQL 엔터프라이즈 에디션에서만 사용 가능하다.

<br>

MySQL 8.0부터는 Caching SHA-2 Authentication이 기본 인증으로 바뀌었다.  
```mysql
-- Native Authentication을 기본 인증 방식으로 설정하기
SET GLOBAL default_authentication_plugin="mysql_native_password"
```

<br>

#### REQUIRE
MySQL 서버에 접속할 때 암호화된 SSL/TLS 채널을 사용할지 여부를 설정한다.  
하지만 `REQUIRE` 옵션을 SSL로 설정하지 않았다고 하더라도 `Caching SHA-2 Authentication` 인증 방식을 사용하면 암호화된 채널만으로 MySQL 서버에 접속할 수 있게 된다.

<br>

#### PASSWORD EXPIRE
비밀번호의 유효 기간을 설정하는 옵션이며, 별도로 명시하지 않으면 `default_password_lifetime` 시스템 변수에 저장된 기간이 유효 기간으로 설정된다.  
개발자나 데이터베이스 관리자의 비밀번호는 유효기간을 설정하는 것이 보안상 안전하지만, 응용 프로그램 접속용 계정에 유효 기간을 설정하는 것은 위험할 수 있으니 조심하자.

- `PASSWORD EXPIRE`: 계정 생성과 동시에 비밀번호의 만료 처리
- `PASSWORD EXPIRE NEVER`: 계정 비밀번호의 만료 기간 없음
- `PASSWORD EXPIRE DEFAULT`: `default_password_lifetime` 시스템 변수에 저장된 기간이 유효 기간으로 설정
- `PASSWORD EXPIRE INTERVAL n DAY`: 비밀번호의 유효 기간을 오늘부터 n일자로 설정

<br>

#### PASSWORD HISTORY
한 번 사용했던 비밀번호를 재사용하지 못하게 설정하는 옵션인데, PASSWORD HISTORY 절에 설정 가능한 옵션은 다음과 같다.

- `PASSWORD HISTORY DEFAULT`: `password_history` 시스템 변수에 저장된 개수만큼 비밀번호의 이력을 저장하며, 저장된 이력에 남아있는 비밀번호는 재사용할 수 없다.
- `PASSWORD HISTORY n`: 비밀번호의 이력을 최근 n개까지만 저장하며, 저장된 이력에 남아있는 비밀번호는 재사용할 수 없다.

```mysql
SELECT * FROM mysql.password_history;
```

<br>

#### PASSWORD REUSE INTERVAL
한 번 사용했던 비밀번호의 재사용 금지 기간을 설정하는 옵션이며, 별도로 명시하지 않으면 `password_reuse_interval` 시스템 변수에 저장된 기간으로 설정된다.  

- `PASSWORD REUSE INTERVAL DEFAULT`: `password_reuse_interval` 변수에 저장된 기간으로 설정
- `PASSWORD REUSE INTERVAL n DAY`: n일자 이후에 비밀번호를 재사용할수 있게 설정

<br>

#### PASSWORD REQUIRE
비밀번호가 만료되어 새로운 비밀번호로 변경할 때 현재 비밀번호(변경하기 전 만료된 비밀번호)를 필요로 할지 말지를 결정하는 옵션으로, 별도로 명시되지 않으면 `password_require_current` 시스템 변수의 값으로 설정된다.

- `PASSWORD REQUIRE CURRENT`: 비밀번호를 변경할 때 현재 비밀번호를 먼저 입력하도록 설정
- `PASSWORD REQUIRE OPTIONAL`: 비밀번호를 변경할 때 현재 비밀번호를 입력하지 않아도 되도록 설정
- `PASSWORD REQUIRE DEFAULT`: `password_require_current` 시스템 변수의 값으로 설정

<br>

#### ACCOUNT LOCK / UNLOCK
계정 생성 시 `ALTER USER` 명령을 사용해 계정 정보를 변경이 가능하게 할지 말지를 결정한다.
- `ACCOUNT LOCK`
- `ACCOUNT UNLOCK`

<br>

## ✅ 비밀번호 관리
<hr>

- [💡 고수준 비밀번호](#-고수준-비밀번호)
- [💡 이중 비밀번호](#-이중-비밀번호)
### 💡 고수준 비밀번호
MySQL 서버의 비밀번호는 유효기간이나 이력 관리를 통한 재사용 금지 기능뿐만 아니라 비밀번호를 쉽게 유추할 수 있는 단어들이 사용되지 않게 글자의 조합을 강제하거나 금칙어를 설정하는 기능도 있다.  
➡ `validate_password` 컴포넌트를 이용해야 하는데, MySQL 서버 프로그램에 내장돼있기 때문에 `file://` 부분에 별도의 파일 경로를 지정하지 않아도 된다.

```mysql
mysql> INSTALL COMPONENT 'file://component_validate_password';
Query OK, 0 rows affected (0.51 sec)

mysql> SELECT * FROM mysql.component;
+--------------+--------------------+------------------------------------+
| component_id | component_group_id | component_urn                      |
+--------------+--------------------+------------------------------------+
|            1 |                  1 | file://component_validate_password |
+--------------+--------------------+------------------------------------+
1 row in set (0.01 sec)
             
mysql> SHOW GLOBAL VARIABLES LIKE 'validate_password%';
+-------------------------------------------------+--------+
| Variable_name                                   | Value  |
+-------------------------------------------------+--------+
| validate_password.changed_characters_percentage | 0      |
| validate_password.check_user_name               | ON     |
| validate_password.dictionary_file               |        |
| validate_password.length                        | 8      |
| validate_password.mixed_case_count              | 1      |
| validate_password.number_count                  | 1      |
| validate_password.policy                        | MEDIUM |
| validate_password.special_char_count            | 1      |
+-------------------------------------------------+--------+
8 rows in set (0.02 sec)
```

<br>

- `validate_password.policy`
  - LOW: 비밀번호의 길이만 검증
  - MEDIUM: 비밀번호의 길이를 검증하며, 숫자와 대소문자, 그리고 특수문자의 배합을 검증
  - STRONG: MEDIUM 레벨의 검증을 모두 수행하며, 금칙어가 포함됐는지 여부까지 검증
- `validate_password_file`: 금칙어들이 저장된 사전 파일을 등록하는 시스템 변수
  - 참고하고 싶다면 [깃허브 저장소의 파일](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/10k-most-common.txt)을 살펴보면 편하다.

<br>

### 💡 이중 비밀번호
MySQL 8.0 버전부터는 이중 비밀번호(Dual Password)를 제공한다.  
- Primary 비밀번호: 최근에 설정된 비밀번호
- Secondary 비밀번호: 이전 비밀번호

<br>

```mysql
#'old_password'로 비밀번호를 설정하기 위해서는 정책이 LOW여야 한다. 
mysql> SET GLOBAL validate_password.policy=LOW;
Query OK, 0 rows affected (0.00 sec)

# old_password로 비밀번호 설정
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'old_password';
Query OK, 0 rows affected (0.02 sec)

# new_password로 비밀번호를 설정하면서 기존 비밀번호를 세컨더리 비밀번호로 설정
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password' RETAIN CURRENT PASSWORD;
Query OK, 0 rows affected (0.02 sec)

# MySQL 서버에 접속하는 모든 응용 프로그램의 소스코드나 설정 파일의 비밀번호를 새로운 비밀번호로 변경 후 배포 및 재시작 수행

# 계정의 보안을 위해 세컨더리 비밀번호 삭제
mysql> ALTER USER 'root'@'localhost' DISCARD OLD PASSWORD;
Query OK, 0 rows affected (0.00 sec)
```

<br>

## ✅ 권한(Privilege)
<hr>

- 글로벌 권한
  - 데이터베이스나 테이블 이외의 객체에 적용되는 권한
  - `GRANT` 명령에서 특정 객체를 명시하지 말아야 한다.
- 객체 권한
  - 데이터베이스나 테이블을 제어하는 데 필요한 권한
  - `GRANT` 명령으로 권한을 부여할 때 반드시 특정 객체를 명시해야한다.
- `ALL`
  - 특정 객체에 `ALL` 권한이 부여되면 해당 객체에 적용될 수 있는 모든 객체 권한을 부여한다.
  - 글로벌로 `ALL`이 사용되면 글로벌 수준에서 가능한 모든 권한을 부여하게 된다.

> 참고로 MySQL 8.0 버전부터는 존재하지 않는 사용자에 대해 `GRANT` 명령이 실행되면 에러가 발생되므로 반드시 사용가가 먼저 생성되어있는 상태여야 한다.

<br>

```mysql
# 글로벌 권한
GRANT privilege_list ON db.table TO 'user'@'host';

# DB(DB 내부에 존재하는 테이블뿐 아니라 스토어드 프로그램들 모두 포함) 권한
GRANT SUPER ON *.* TO 'user'@'localhost';
GRANT SUPER ON employees.* TO 'user'@'localhost';
```
글로벌 권한은 특정 DB나 테이블에 부여될 수 없기 때문에 글로벌 권한은 부여할 때 `GRANT` 명령의 `ON` 절에는 항상 `*.*`를 사용하게 된다.

<br>

```mysql
# DB 권한
GRANT EVENT ON *.* TO 'user'@'localhost';
GRANT EVENT ON employees.* TO 'user'@'localhost';
```
DB 권한은 서버의 모든 DB에 적용할 수 있기 때문에 `*.*`을 사용할 수 있다.  
또한 특정 DB에 대해서만 권한을 부여하는 것도 가능하기 때문에 `db.*`로 대상을 설정하는 것도 가능하다.  
하지만 오브젝트 권한처럼 `db.table`로 오브젝트(테이블)까지 명시할 수는 없다.

<br>

```mysql
# 테이블 권한
GRANT SELECT,INSERT,UPDATE,DELETE ON *.* TO 'user'@'localhost';
GRANT SELECT,INSERT,UPDATE,DELETE ON employees.* TO 'user'@'localhost';
GRANT SELECT,INSERT,UPDATE,DELETE ON employees.department TO 'user'@'localhost';
```


<br>

## ✅ 역할(Role)
<hr>

```mysql
# 역할 정의
CREATE ROLE
    role_emp_read,
    role_emp_write;

# role_emp_read 역할에는 employees DB의 모든 객체에 대한 읽기 권한 부여
GRANT SELECT ON employees.* TO role_emp_read;
# role_emp_write 역할에는 employees DB의 모든 객체에 대한 변경 권한 부여
GRANT INSERT, UPDATE, DELETE ON employees.* TO role_emp_write;


# reader 계정 생성
CREATE USER writer@'127.0.0.1' IDENTIFIED BY 'password';
Query OK, 0 rows affected (0.01 sec)

# writer 계정 생성
CREATE USER reader@'127.0.0.1' IDENTIFIED BY 'password';
Query OK, 0 rows affected (0.01 sec)
```



<br>

**참고 자료**  
[Real MySQL 8.0](https://product.kyobobook.co.kr/detail/S000001766482)