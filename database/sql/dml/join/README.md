# 조인
```sql
SELECT A.컬럼1, A.컬럼2, ..., B.컬럼1, B.컬럼2, ...
FROM 테이블1 A [INNER] JOIN 테이블2 B 
ON 조인조건
[WHERE 검색조건];
```

### 예시
```sql
SELECT A.책번호 AS 책번호, A.책명 AS 책명, B.가격 AS 가격
FROM 도서 A JOIN 도서가격 B
ON A.책번호 = B.책변호;
```

### 다른 조인들
- 내부조인: `[INNER] JOIN`
- 오른쪽 외부 조인: `RIGHT [OUTER] JOIN`
- 완전 외부 조인: `FULL [OUTER] JOIN`
- 교차 조인: `CROSS JOIN`

사용하는 구조는 모두 같다.