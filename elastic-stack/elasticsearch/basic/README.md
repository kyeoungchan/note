# 💻 엘라스틱서치 기본
> Mac 환경과 Elasticsearch 9.1.2 버전으로 기록하였습니다.


### ✅ 시스템 상태 확인
Kibana 콘솔 창에서 `GET _cat/indices?v` 를 입력하면 현재 클러스터에서의 인덱스들 상태를 확인할 수 있다.  
파라미터는 다음과 같다.
- v: 컬럼 헤더 확인
- s: 정렬
- h: 헤더

<br>

Terminal 창에서도 `curl` 명령어를 통해 호출할 수 있다.  
```text
curl -kX GET "https://localhost:9200/_cat/indices?v" -u elastic:???
```

<br>

> 참고로 -k 옵션이 있어야 일시적으로 Mac의 curl 보안을 우회할 수 있다.  
> elasticsearch 8.0 버전부터는 기본적으로 SSL/TLS가 활성화되어 "https"로 통신해야한다.   
> 그리고 ???에는 elasticsearch의 따로 저장한 비밀번호를 입력한다.  
> → 안 그러면 `missing authentication credentials for REST request` 에러가 난다.  

<br>

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

<br>

> 참고로 -k 옵션이 있어야 일시적으로 Mac의 curl 보안을 우회할 수 있다.  
> elasticsearch 8.0 버전부터는 기본적으로 SSL/TLS가 활성화되어 "https"로 통신해야한다.  
> 그리고 ???에는 elasticsearch의 따로 저장한 비밀번호를 입력한다.  
> → 안 그러면 `missing authentication credentials for REST request` 에러가 난다.

<br>

그러면 다음과 같은 결과가 나온다.

```text
{"errors":false,"took":0,"items":[{"index":{"_index":"index2","_id":"6","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":7,"_primary_term":2,"status":201}},{"index":{"_index":"index2","_id":"7","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":8,"_primary_term":2,"status":201}}]}
```

<br>

그리고 Kibana에서 `GET index2/_search` 명령어를 통해 확인해보면 데이터가 추가로 인덱싱된 것을 확인할 수 있다.

<br>

### ✅ 명시적 매핑
직접 매핑을 지정하여 인덱스 생성
```shell
# index3를 만들면서 직접 매핑
PUT index3
{
    "mappings": {
        "properties": {
            "age": {"type": "short"},
            "name": {"type": "text"},
            "gender": {"type": "keyword"}
        }
    }
}
```

<br>

```shell
# index3 인덱스 매핑 결과
GET index3/_mapping
```
```shell
{
  "index3": {
    "mappings": {
      "properties": {
        "age": {
          "type": "short"
        },
        "gender": {
          "type": "keyword"
        },
        "name": {
          "type": "text"
        }
      }
    }
  }
}
```

<br>

```shell
# 타입을 맞지 않게 도큐먼트 생성 시도 시
PUT index3/_doc/1
{
    "age": "20 years old",
    "country": "korea"
}
```
```shell
{
  "error": {
    "root_cause": [
      {
        "type": "document_parsing_exception",
        "reason": "[1:13] failed to parse field [age] of type [short] in document with id '1'. Preview of field's value: '20 years old'"
      }
    ],
    "type": "document_parsing_exception",
    "reason": "[1:13] failed to parse field [age] of type [short] in document with id '1'. Preview of field's value: '20 years old'",
    "caused_by": {
      "type": "number_format_exception",
      "reason": "For input string: \"20 years old\""
    }
  },
  "status": 400
}
```

<br>

```shell
# 새로운 필드는 그냥 추가가 된다.
PUT index3/_doc/1
{
    "country": "korea"
}
```
```shell
{
  "_index": "index3",
  "_id": "1",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

<br>

```shell
GET index3/_mapping
```
```shell
{
  "index3": {
    "mappings": {
      "properties": {
        "age": {
          "type": "short"
        },
        "country": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "gender": {
          "type": "keyword"
        },
        "name": {
          "type": "text"
        }
      }
    }
  }
}
```

<br>

**참고 자료**  
[엘라스틱 스택 개발부터 운영까지](https://product.kyobobook.co.kr/detail/S000001932755)