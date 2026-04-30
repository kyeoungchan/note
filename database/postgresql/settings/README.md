# 🧑🏻‍💻 PostgreSQL 설정 관련 해결하기 - Mac OS

---

## ✅ Docker

---

```shell
docker run -e POSTGRES_PASSWORD={password} -p 5432:5432 -v {volume-name}:/var/lib/postgresql -d postgres:17

docker exec -it {container_id} bash

psql -U postgres

# 데이터베이스 목록 확인
\l

# db 접속
\c mydb

# 테이블 목록 확인
\dt

# psql 쉘 종료
\q
```