# 데이터베이스 핸들링 SQL문
```sql
-- 모든 데이터베이스 조회
SHOW DATABASES;

-- 데이터베이스 생성
CREATE DATABASE {데이터베이스 이름} DEFAULT CHARACTER SET utf8 COLLATE utf_8_general_ci;
       
-- 데이터베이스 삭제
DROP DATABASE IF EXISTS {데이터베이스 이름};
```