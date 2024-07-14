# 트랜잭션
트랜잭션은 데이터베이스 시스템에서 **하나의 논리적 기능**을 정상적으로 수행하기 위한 **작업의 기본 단위**
## 트랜잭션의 주요 특성
- 원자성(Atomicity)
  - 분해가 불가능한 작업의 최소단위
  - 연산 전체가 성공 또는 실패(All or Nothing)
  - 하나라도 실패할 경우 전체가 취소되어야 하는 특성
- 일관성(Consistency)
  - 트랜잭션이 실행 성공 후 항상 일관된 데이터베이스 상태를 보존해야하는 특성
- 격리성(Isolation)
  - 트랜잭션 실행 중 생성하는 연산의 중간 결과를 다른 트랜잭션이 접근 불가한 특성
- 영속성(Durability)
  - 성공이 완료된 트랜잭션의 결과는 영속적으로 데이터베이스에 저장하는 특성