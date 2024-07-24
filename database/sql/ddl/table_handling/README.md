# 테이블 핸들링
```sql
use mysql;
-- 테이블 조회하기
SHOW TABLES;

-- 테이블 구조 확인하기
DESC {테이블명};

-- 테이블 생성하기
CREATE TABLE member_table (
    seq        INT NOT NULL AUTO_INCREMENT,
    mb_id     VARCHAR(20),
    mb_pw    VARCHAR(20),
    address   VARCHAR(50),
    mb_tell    VARCHAR(50),
    PRIMARY KEY(seq)
) ENGINE=MYISAM CHARSET=utf8;

CREATE TABLE professor
(
    _id INT AUTO_INCREMENT,
    name VARCHAR(32) NOT NULL,
    belong VARCHAR(12) DEFAULT 'FOO',
    phone VARCHAR(12),
    PRIMARY KEY(_id)
) ENGINE=INNODB;
```

> InnoDB vs. MyISAM
> - MyISAM
>   - MySQL 5.5.4까지의 디폴트 엔진
>   - 테이블에 Row Count를 가지고 있어 COUNT(*)가 빠르다.
>   - 풀텍스트 인덱스를 지원해 SELECT 명령 시에도 빠른 속도를 자랑한다.
>   - Row 단위 락을 지원하지 않아서 SELECT, INSERT, UPDATE, DELETE 시 테이블 전체가 락이 걸린다.
>   - Row 수가 커질수록 속도는 엄청나게 느리다.
>   - 트랜잭션을 지원하지 않아 잘못 처리된 데이터는 롤백할 수 없다.
> - InnoDB
>   - MySQL 5.5.5 이후 디폴트 엔진
>   - 트랜잭션 기능이 부가되어 MyISAM을 대체하는 엔진으로 사용된다.
>   - Row 단위 락, FK를 지원한다.
>   - 대용량 데이터를 처리할 때 최대의 퍼포먼스를 내도록 설계되었다.
>   - COUNT(*)를 하는 것이 느리다.
>   - 5.5.6 이후부터는 풀 텍스트 인덱스를 지원해 SELECT 명령시에도 속도가 느리지 않다.

출처  
[MySQL - 테이블 생성 알아보기 - CREATE TABLE](https://server-talk.tistory.com/279)  
[(MySQL) 1장 시작하기. (DB 생성, 테이블 생성, SELECT)](https://futurists.tistory.com/11)