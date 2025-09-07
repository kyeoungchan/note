# 💻 유사도 스코어
쿼리 컨텍스트는 엘라스틱에서 지원하는 다양한 스코어 알고리즘을 사용할 수 있는데 기본적으로 BM25 알고리즘을 이용해 유사도 스코어를 계산한다.  

<br>

스코어가 어떤식으로 계산되었는지 알아보기 위해서는 쿼리에 `explain:true` 옵션을 추가하면 된다.  
```shell
# 설명이 포함된 쿼리 컨텍스트 실행
GET kibana_sample_data_ecommerce/_search
{
    "query": {
        "match": {
          "products.product_name": "Pants"
        }
    },
    "explain": true
}
```

<br>

➡️ 결과  
```shell
{
  ...
  "hits":
  } {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": 8.268259,
    "hits": [
      {
        ...
        "_score": 8.268259,
        "_source":
        } {
          "category": [
            "Men's Shoes",
            "Men's Clothing"
          ],
          ...
          "products": [
            {
              ...
              "product_name": "Boots - tan",
              ...
            },
            {
              ...
              "product_name": "Casual Cuffed Pants",
              ...
            }
          ],
          ...
        },
        "_explanation": {
          "value": 8.268259,
          "description": "weight(products.product_name:pant in 594) [PerFieldSimilarity], result of:",
          "details": [
            {
              "value": 8.268259,
              "description": "score(freq=1.0), computed as boost * idf * tf from:",
              "details": [
                {
                  "value": 2.2,
                  "description": "boost",
                  "details": []
                },
                {
                  "value": 7.1974354,
                  "description": "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
                  "details": [
                    {
                      "value": 3,
                      "description": "n, number of documents containing term",
                      "details": []
                    },
                    {
                      "value": 4675,
                      "description": "N, total number of documents with field",
                      "details": []
                    }
                  ]
                },
                {
                  "value": 0.52217203,
                  "description": "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
                  "details": [
                    {
                      "value": 1,
                      "description": "freq, occurrences of term within document",
                      "details": []
                    },
                    {
                      "value": 1.2,
                      "description": "k1, term saturation parameter",
                      "details": []
                    },
                    {
                      "value": 0.75,
                      "description": "b, length normalization parameter",
                      "details": []
                    },
                    {
                      "value": 5,
                      "description": "dl, length of field",
                      "details": []
                    },
                    {
                      "value": 7.3161497,
                      "description": "avgdl, average length of field",
                      "details": []
                    }
                  ]
                }
              ]
            }
          ]
        }
      },
...
```
총 3개의 도큐먼트를 찾았고, 그 중에서 가장 높은 스코어의 값이 8.268259이다.  
스코어가 높을수록 찾고자하는 도큐먼트에 가깝다는 뜻이다.

<br>

## ✅ 스코어 알고리즘(BM25) 이해하기
> 엘라스틱서치 5.x 이전 버전에서는 TF-IDF 알고리즘을 사용했다.  
> 이후부터는 TF(Term Frequency), IDF(Inverse Document Frequency) 개념에 문서 길이를 고려한 알고리즘인 BM25 알고리즘을 사용한다.

<br>

### 💡 IDF 계산
문서 빈도(Document Frequency)는 전체 문서에서 얼마나 자주 등장했는가를 나타내는 지표다.  
전체 문서에서 자주 발생할수록 'to', 'the' 같은 관사나, '그리고', '그러나' 같은 접속부사일 확률이 높기 때문에 문서 빈도는 적을수록 가중치를 높게 준다.  
이를 **문서 빈도의 역수(Inverse Document Frequency)** 라고 한다.

<br>

IDF 계산하는 식은 위에 보면 알다시피 엘라스틱서치가 굉장히 친절하게 알려주고 있다.  

```shell
{
  "value": 7.1974354,
  "description": "idf, computed as log(1 + (N - n + 0.5) / (n + 0.5)) from:",
  "details": [
    {
      "value": 3,
      "description": "n, number of documents containing term",
      "details": []
    },
    {
      "value": 4675,
      "description": "N, total number of documents with field",
      "details": []
    }
  ]
}
```

1. 총 3개의 도큐먼트에서 `product.product_name` 필드에 'Pants'라는 용어를 포함하고 있다.
2. `kibana_sample_data_ecommerce` 인덱스는 총 4,675개의 도큐먼트를 갖고 있다.

<br>

### 💡 TF 계산
용어 빈도(Term Frequency)는 특정 용어가 하나의 도큐먼트에 얼마나 등장했는지를 의미한다.  
하나의 도큐먼트에서 특정 용어가 많이 나오면 중요한 용어로 인식하고 가중치를 높인다.  

```shell
{
  "value": 0.52217203,
  "description": "tf, computed as freq / (freq + k1 * (1 - b + b * dl / avgdl)) from:",
  "details": [
    {
      "value": 1,
      "description": "freq, occurrences of term within document",
      "details": []
    },
    {
      "value": 1.2,
      "description": "k1, term saturation parameter",
      "details": []
    },
    {
      "value": 0.75,
      "description": "b, length normalization parameter",
      "details": []
    },
    {
      "value": 5,
      "description": "dl, length of field",
      "details": []
    },
    {
      "value": 7.3161497,
      "description": "avgdl, average length of field",
      "details": []
    }
  ]
}
```
1. `freq`: 도큐먼트 내에서 용어가 나온 횟수 → 1번 나왔다.
2. `k1`, `b`: 알고리즘을 정규화하기 위한 가중치로, 엘라스틱서치가 디폴트로 취하는 상수다.
3. `dl`은 필드 길이, `avgdl`은 전체 도큐먼트에서 평균 필드 길이다.
    - `dl`이 작고, `avgdl`이 클수록 TF 값이 크게 나온다.
    - 짧은 글에서 찾고자하는 용어가 포함될수록 가중치가 높다는 뜻이다.

<br>

```shell
  "products": [
    {
      ...
      "product_name": "Boots - tan",
      ...
    },
    {
      ...
      "product_name": "Casual Cuffed Pants",
      ...
    }
  ],
```
`dl`을 계산하자면, 위의 도큐먼트는 5개의 토큰으로 나뉜다.(Boots, tan, Casual, Cuffed, Pants) ➡️ `dl` = 5  
`avgdl`은 `dl`과 마찬가지로 인덱스 내의 모든 도큐먼트(4,675개)의 평균 토큰 수다. ➡️ `avgdl` = 7.3161497 (엘라스틱서치가 계산해줌)  

<br>

### 💡 최종 계산
`boot`는 엘라스틱이 지정한 고정 값으로, 2.2로 나와있다.  

```shell
  "description": "score(freq=1.0), computed as boost * idf * tf from:",
  "details": [
    {
      "value": 2.2,
      "description": "boost",
      "details": []
    },
```

이제 지금까지 구한 `boost`, `idf`, `tf`를 곱하면 스코어가 나온다.  


<br>

**참고 자료**  
[엘라스틱 스택 개발부터 운영까지](https://product.kyobobook.co.kr/detail/S000001932755)