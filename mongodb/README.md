# 🧑🏻‍💻 MongoDB
<hr>

- [✅ MongoDB vs RDBMS(MySQL)](#-mongodb-vs-rdbmsmysql)

> [!NOTE]
> 예전에는 데이터베이스는 관계형 데이터베이스와 동의어로 사용될 정도로 대부분 기업용 데이터베이스 관리 시스템은 관계형 데이터베이스(Relational DBMS)가 중심이었다.  
> 하지만 방대한 양의 데이터를 충분히 빠른 속도로 처리할 수 있는 데이터베이스에 대한 필요성이 대두되기 시작했다.  
> 기존의 RDBMS에서 하나의 테이블에 저장된 데이터를 수십에서 수백 대의 서버로 쪼개야만 처리할 수 있을 정도로 커져 버렸는데, 이렇게 많은 서버에 오라클 RDBMS를 설치한다면 엄청난 라이선스 비용에 시달리게 될 것이 불 보듯 뻔했기 때문이다.

> [!TIP]
> MySQL 서버가 단기적으로는 유일한 해결책이었다.  
> MySQL은 오픈 소스 데이터베이스이므로 별도의 라이선스 비용이 없을 뿐만 아니라, 기술력을 가진 글로벌 기업들에겐느 더없이 좋은 해결책이었다.  
> 얼마든지 필요한 기능들을 자체적으로 구현해서 사용하고, 그와 동시에 오픈 소스 커뮤니티로부터 얻은 버그 리포트와 성능 이슈들을 활용할 수 있기 때문이다.

> [!IMPORTANT]
> MongoDB는 다른 NoSQL과는 지향점이나 적용 가능 범위가 조금 다르다.  
> MongoDB는 온라인 서비스에 필요한 블록 캐시와 보조 인덱스 그리고 동시성 처리를 위한 스킵리스트(Skip-List)나 하자드 포인터(Hazard Pointer)와 같은 특성들을 많이 가지고 있다.  
> MongoDB가 WiredTiger 스토리지 엔진은 MySQL 서버의 InnoDB 스토리지 엔진만큼 성능과 안전성을 갖추게 될 것이다.

<br>

## ✅ MongoDB vs RDBMS(MySQL)
<hr>

| MongoDB                 | RDBMS(MySQL)             |
|-------------------------|--------------------------|
| 데이터베이스(Database)        | 데이터베이스(Database)         |
| 컬렉션(Collection)         | 테이블(Table)               |
| 도큐먼트(Document)          | 레코드(Record 또는 Row)       |
| 필드(Field)               | 컬럼(Column)               |
| 인덱스(Index)              | 인덱스(Index)               |
| 쿼리의 결과로 "커서(Cursor)" 반환 | 쿼리의 결과로 "레코드(Record)" 반환 |


> [!TIP]
> MongoDB는 쿼리 결과로 커서를 반환하는데, 응용 프로그램이나 MongoDB 클라이언트 프로그램(Mongo 셸)에서 커서를 통해 반복적으로 실제 도큐먼트(레코드)를 가져올 수 있다.  
> ➡️ 쿼리의 결과를 커서로 반환함으로써 쿼리의 결과를 클라이언트 서버의 메모리에 담아두지 않아도 처리할 수 있게 한다.  
> ❗️ 물론 MongoDB에서 커서를 읽을 때마다 서버에서 그때그때 도큐먼트를 가져오는 것은 아니고, 필요할 때마다 지정된 페이지 사이즈 단위로 서버로부터 전송받아 MongoDB 클라이언트 서버에 캐싱한 후에 유저에게 서비스하는 것이다.

> [!IMPORTANT]
> MongoDB는 외래키를 명시적으로 지원하지는 않지만, `$lookup`이라는 Aggregation 기능을 이용하면 RDBMS와 비슷한 형태의 조인 처리를 수행(샤딩 환경에서는 여러 제약이 있지만)할 수 있다.

> [!IMPORTANT]
> Schema-Free가 아마도 MongoDB와 RDBMS를 구분 지어줄 수 있는 가장 좋은 단어가 아닐까 생각된다.  
> 스키마 프리는 사용할 컬럼을 미리 정의하지 않고 언제든지 필요한 시점에 데이터를 저장할 수 있다는 것을 의미한다.  
> 대부분의 NoSQL 데이터베이스는 컬럼의 메타 정보(컬럼의 이름과 데이터 타입 등)를 컬럼 값과 똑같이 사용자 데이터로 관리하므로 이런 자유로운 스키마 구성이 가능하다.  
> 하지만 MongoDB는 다른 NoSQL 데이터베이스와는 달리 보조 인덱스를 생성할 수 있는데, MongoDB의 보조 인덱스는 스키마 프리가 아니라 항상 먼저 인덱스를 구성하는 필드를 정의해야 한다.

| MongoDB                                                                                            | MySQL                                                                    |
|----------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| db.users.insert({ <br> user_id: 'bcd001', <br> age:45, <br> status: 'A' <br> })                    | INSERT INTO users (user_id, age, status) <br> VALUES ('bcd001', 45, 'A') |
| db.users.update( <br> { age: {$gt: 25} }, <br> {`$`set: {status: 'C'} }, <br> {multi: true} <br> } | UPDATE users <br> SET status='C' <br> WHERE age > 25                     |
| db.users.remove( <br> { user_id : 'bcd001' } <br> }                                                | DELETE <br> FROM users <br> WHERE user_id='bcd001'                       |
| db.users.find( <br> { user_id : 'bcd001' } <br> }                                                  | SELECT * <br> FROM users <br> WHERE user_id='bcd001'                      |

<br>

## ✅ MongoDB vs NoSQL(HBase)
<hr>

> [!IMPORTANT]
> MongoDB와 HBase는 기본적인 데이터 저장 포맷에서 큰 차이가 있다.  
> HBase는 저장소 포맷으로 구분할 때 주로 컬럼 스토어(Column Store)라는 표현을 사용하지만, 더 정확하게는 컬럼 패밀리 스토어(Column Family Store)라고 볼 수 있다.  
> ➡️ 즉, 각 컬럼을 별도의 파일로 저장하는 것이 아니라, 하나의 테이블 안에서도 관련된 컬럼들을 묶어서 여러 개의 컬럼 패밀리를 만들고, 이 컬럼 패밀리 단위로 데이터 파일을 생성하여 관리하게 된다.  
> 하지만 MongoDB는 컬럼이나 컬럼 패밀리 단위의 그룹 개념이 없으며, 한 테이블의 데이터(Document)는 하나의 데이터 파일로 저장된다.

<br>

#### HBase
| RowKey | Column Family | Column Qualifier | Timestamp  | Value                       |
|--------|---------------|------------------|------------|-----------------------------|
| 1234   | basic         | name             | 1368394583 | "홍길동"                       |
| 1234   | basic         | joined_at        | 1368394583 | "2026-06-26 07:49:25"       |
| 1234   | relationship  | contacts         | 1368394583 | "000-000-0000,111-111-1111" |
| 1234   | relationship  | friends          | 1368394583 | "123,187,291,221"           |
| 1234   | security      | role             | 1368394583 | "Administrator"             |
| 1234   | security      | permission       | 1368394583 | "Create,Modify,Remove"      |

<br>

#### MongoDB
```json
{
  "name": "홍길동",
  "joined_at": "2026-06-26 07:49:25",
  "relationship": {
    "contacts": [ "000-000-0000, 111-111-1111" ],
    "friends": [123, 187, 291, 221]
  },
  "security": {
    "role": "Administrator",
    "permission": ["Create, Modify, Remove"]
  }
}
```

<br>

> [!TIP]
> MongoDB에서는 로우 키라는 표현보다는 도큐먼트 Id라는 표현이 일반적이며, 도큐먼트에서 "_id"라는 이름의 필드가 자동으로 그 도큐먼트의 프라이머리 키로 선정된다.  
> MongoDB에서는 Value의 JSON 값을 문자열로 그대로 저장하는 것이 아니라 문자열 기반의 JSON 텍스트를 BSON(Binary JSON)으로 변환해서 저장하므로, 공백이나 마크업(Mark-up) 문자로 인해 부가적으로 저장 공간이 낭비되지 않는다.



<br>

**참고 자료**  
[대용량 데이터 처리를 위한 Real MongoDB](https://product.kyobobook.co.kr/detail/S000001766322)