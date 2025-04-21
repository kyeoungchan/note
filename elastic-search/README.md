# 💻 Elastic Search
> 모든 검색엔진의 시초는 루씬(Lucene)이었지만, 현재는 루씬을 기반으로 한 Elastic Search가 등장해 검색엔진 분야에서 지배적인 위치에 있다.

## ✅ Elastic Search와 RDMS
조금 더 쉬운 개념 파악을 위해서 ES에서 사용되는 데이터 구조를 RDBM에 대응해보면 다음과 같이 매핑된다.

|ES| RDBMS  |
|---|--------|
|Index|Database|
|Shard|Position|
|Type|Table|
|Document|Row|
|Field|Column|
|Mapping|Schema|
|QueryDSL|SQL|

Elastic Search는 기본적으로 HTTP 프로토콜로 접근이 가능한 REST API를 통해 데이터 조작을 지원한다.  
이를 역시 RDBMS의 SQL과 매핑해본다면 다음과 같다.

|ES HTTP Method|RDBMS SQL|
|---|---|
|GET|SELECT|
|PUT|INSERT|
|POST|UPDATE, SELECT|
|DELETE|DELETE|
|HEAD(인덱스 정보 확인)|

<br>

## ✅ 역색인
일반적인 색인의 목적은 '문서의 위치'에 대한 index를 만들어서 빠르게 그 문서에 접근하고자 하는 것인데, 역색인은 반대로 '문서 내의 문자와 같은 내용물'의 매핑 정보를 색인해놓는 것이다.  
역색인 검색엔진과 같은 문서의 내용의 검색이 필요한 형태에서 전문 검색의 형태로 주로 쓰인다.  
쉬운 예시로 들어보자면 **일반 색인(forward index)은 책의 목차**와 같은 의미이고, **역색인(inverted index)은 책 가장 뒤의 단어 별 색인 페이지**와 같다.  

## ✅ ES의 특징과 장단점


<br>

**출처**  
[[Elastic Search] 기본 개념과 특징(장단점)](https://jaemunbro.medium.com/elastic-search-%EA%B8%B0%EC%B4%88-%EC%8A%A4%ED%84%B0%EB%94%94-ff01870094f0)