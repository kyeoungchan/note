# 로킹기법(Locking)
- 로킹 기법이란 트랜잭션이 사용하는 데이터 항목에 대하여 잠금(Lock)을 설정한 트랜잭션이 해제(Unlock)할 때까지 독점적으로 사용할 수 있게 상호배제 기능을 제공하는 기법이다.
- [병쟁제어 기법](https://github.com/kyeoungchan/note/tree/main/database/concurrency-control) 중의 하나로 그 외에는 다음과 같이 있다.
  - 낙관적 검증(Validation) 기법
  - 타임 스탬프 순서(Timestamp ordering) 기법
  - 다중버전 동시성 제어(MVCC) 기법
