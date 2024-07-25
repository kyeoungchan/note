# 테이블 수정하기
```sql
ALTER TABLE {테이블명} MODIFY {컬럼명} {데이터타입}[제약조건];
```
예를 들어, 다음과 같이 작성할 수 있다.
```sql
ALTER TABLE 부서 MODIFY 부서번호 INTEGER PRIMARY KEY;
```
외래키를 설정하는 방식도 있다. 링크: [Foreign Key](https://github.com/kyeoungchan/note/tree/main/database/sql/ddl/foreign-key)

