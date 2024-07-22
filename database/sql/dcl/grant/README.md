# Grant
```sql
GRANT 권한 ON 테이블 TO 사용자;
```
예를 들어, 다음과 같이 작성할 수 있다.
```sql
GRANT SELECT ON DEPT TO 찰스;
```

## 다른 사람에게도 권한을 부여할 수 있는 권한 부여하기
```sql
GRANT 권한 ON 테이블 TO 사용자 WITH GRANT OPTION;
```
예를 들어, 다음과 같이 작성할 수 있다.
```sql
GRANT ALL ON STUDENT TO SOOJEBI_SYS WITH GRANT OPTION;
```