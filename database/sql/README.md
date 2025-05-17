# 💻 SQL SELECT 쿼리 문법 순서와 실행 순서

SQL의 SELECT 쿼리문이 어떤 순서로 실행이 되는지 알면 올바르고 효율적인 쿼리를 짜는 데에 많은 도움이 된다.  
따라서 SQL 문법 순서와 실제 실행 순서를 다음과 같이 정리해본다.

### ✅ SQL 문법 순서
- SELECT
- FROM
- WHERE
- GROUP BY
- HAVING
- ORDER BY
- LIMIT

### ✅ SQL 실제 실행 순서
- FROM
- ON
- JOIN
- WHERE
- GROUP BY
- HAVING
- SELECT
- DISTINCT
- ORDER BY

> HAVING 절은 그루핑 후에 각 그룹에 사용되는 조건 절이다. 쉽게 말해 그룹을 필터링한다고 생각하면 된다.  
> HAVING 절의 조건을 WHERE 절에도 사용할 수 있는 경우라면 WHERE 절에 사용하는 것이 바람직하다. **HAVING 절은 각 그룹에 조건을 걸기 때문에 퍼포먼스가 떨어지게 된다.**