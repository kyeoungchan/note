# 외래키 제약 조건 설정하기

테이블을 지우려다가 외래키 제약조건에 의해 함부로 테이블을 삭제할 수 없을 때 해결해주는 방법이다.
1. DELTE CASCADE : 부모 레코드를 제거할 때 자식 레코드도 함께 제거한다.
    ```sql
    ALTER TABLE memo ADD CONSTRAINT FOREIGN KEY (user_id) REFERENCES members(user_id) ON DELETE CASCADE;
    ```
2. DELTE SET NULL: 부모 레코드를 삭제하면 자식 레코드의 외래키는 null로 바꿔준다.
   ```sql
    ALTER TABLE memo ADD CONSTRAINT FOREIGN KEY (user_id) REFERENCES members(user_id) ON DELETE SET NULL;
    ```
