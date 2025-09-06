# 💻 엘라스틱서치 기본
> Mac 환경과 Elasticsearch 9.1.2 버전으로 기록하였습니다.

### ✅ 벌크 데이터 파일로 실행하기
원하는 경로에서 파일을 생성한다.
```text
# example 디렉터리 생성 및 이동
mkdir example
cd example

# bulk_index2 파일 생성
touch bulk_index2
```

그리고 해당 파일에 bulk 데이터를 입력한다.
```text
# bulk_index2 파일 수정
vi bulk_index2
```
아래와 같은 내용이 `bulk_index2` 파일에 입력된다.
```text
{"index": {"_index": "index2", "_id": "6"}}
{"name": "hong", "age": 10, "gender": "female"}
{"index": {"_index": "index2", "_id": "7"}}
{"name": "choi", "age": 90, "gender": "male"}
```

<br>

그리고 example 디렉토리에서 아래와 같은 명령어를 입력한다.

```text
curl -kH "Content-Type: application/x-ndjson" -XPOST https://localhost:9200/_bulk --data-binary @./bulk_index2 -u elastic:???
```
> 참고로 -k 옵션이 있어야 일시적으로 Mac의 curl 보안을 우회할 수 있다.  
> 그리고 ???에는 elasticsearch의 따로 저장한 비밀번호를 입력한다.
  
그러면 다음과 같은 결과가 나온다.

```text
{"errors":false,"took":0,"items":[{"index":{"_index":"index2","_id":"6","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":7,"_primary_term":2,"status":201}},{"index":{"_index":"index2","_id":"7","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":8,"_primary_term":2,"status":201}}]}
```

<br>

그리고 Kibana에서 `GET index2/_search` 명령어를 통해 확인해보면 데이터가 추가로 인덱싱된 것을 확인할 수 있다.

<br>

**참고 자료**  
[엘라스틱 스택 개발부터 운영까지](https://product.kyobobook.co.kr/detail/S000001932755)