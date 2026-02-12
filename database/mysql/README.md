# 🧑🏻‍💻 MySQL
## ✅ 왜 MySQL인가?
MySQL과 오라클을 비교해본다면 당연히 MySQL의 경쟁력은 가격이나 비용일 것이다.  
> 페이스북이 가진 데이터를 모두 오라클 RDBMS에 저장하면 페이스북은 망할 것이다.  
> - 페이스북에 근무하는 어느 DBA

### 💡 어떤 RDBMS가 좋은가?
- 안정성
- 성능과 기능
- 커뮤니티나 인지도
  - [DB-Engines.com](https://db-engines.com/en/ranking)에서 RDBMS 서버의 랭킹을 확인할 수 있다.

<br>

## ✅ 엔터프라이즈 에디션 vs 커뮤니티 에디션
초기 버전의 MySQL 서버는 엔터프라이즈 에디션과 커뮤니티 에디션으로 나뉘어 있기는 했지만 실제 MySQL 서버의 기능에 차이가 있었던 것이 아니라 기술 지원의 차이만 있었다.  
하지만 MySQL 5.5 버전부터는 커뮤니티와 엔터프라이즈 에디션의 기능이 달라지면서 소스코드도 달라졌고, MySQL 엔터프라이즈 에디션의 소스코드는 더이상 공개되지 않는다.  

하지만 MySQL 서버의 상용화 전략은 아래와 같은데, 오픈 코어 모델(Open Core Model)이라 한다.
- 핵심 내용은 엔터프라이즈 에디션과 커뮤니티 에디션 모두 동일하다.
- 특정 부가 기능들만 상용 버전인 엔터프라이즈 에디션에 포함된다.

<br>

🤔 엔터프라이즈 에디션에서만 지원되는 기능
- Thread Pool
- Enterprise Audit
- Enterprise TDE(Master Key 관리)
- Enterprise Authentication
- Enterprise Firewall
- Enterprise Monitor
- Enterprise Backup
- MySQL 기술 지원

[Percona Server](https://www.percona.com/)에서 지원하는 플러그인을 사용하면 MySQL 커뮤니티 에디션의 부족한 부분을 메꿀 수 있다.  
물론 기술 지원은 별개의 문제여서, 엔터프라이즈 에디션이 꼭 필요한지 검토할 필요가 있다.


<br>

**참고 자료**  
[Real MySQL 8.0](https://product.kyobobook.co.kr/detail/S000001766482)