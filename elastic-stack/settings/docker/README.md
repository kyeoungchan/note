# 🧑🏻‍💻 docker로 elastic-stack 실행하기
```yml
services:
  # Elasticsearch - 검색 최적화된 쿼리 모델용
  elasticsearch:
    image: elasticsearch:9.1.4
    container_name: charles-elasticsearch
    environment:
      - discovery.type=single-node
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
      - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data

  # Kibana - Elasticsearch 관리 및 모니터링용
  kibana:
    image: kibana:9.1.4
    container_name: charles-kibana
    depends_on:
      - elasticsearch
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=https://elasticsearch:9200
      - ELASTICSEARCH_USERNAME=kibana_system
      - ELASTICSEARCH_PASSWORD=${KIBANA_SYSTEM_PASSWORD}
      # 서비스 토큰을 생성하고 사용한다면 위의 USERNAME과 PASSWORD는 없어도 된다.
      #- ELASTICSEARCH_SERVICE_ACCOUNT_TOKEN=${KIBANA_SERVICE_TOKEN}
```

```shell
# charles-elasticsearch 컨테이너 bash로 접속
docker exec -it charles-elasticsearch bash

# elastic 유저로 패스워드 새로 생성
bin/elasticsearch-reset-password -u elastic

# kibana 패스워드 직접 설정
bin/elasticsearch-reset-password -u kibana_system -i

# kibana 패스워드 자동 생성
bin/elasticsearch-reset-password -u kibana_system

# kibana용 서비스 토큰 생성. 위에 ELASTICSEARCH_SERVICE_ACCOUNT_TOKEN 옵션에 해당
bin/elasticsearch-service-tokens create elastic/kibana kibana-token
```