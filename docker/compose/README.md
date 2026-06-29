# 🧑🏻‍💻 Docker Compose
```yaml
services:
  my-server:
    build: .
    ports:
      - 8081:8080
    depends_on:
      # my-db의 헬스를 체크하고 시작
      my-db:
        condition: service_healthy
      # my-cache-server의 헬스를 체크하고 시작
      my-cache-server:
        condition: service_healthy

  my-db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: pwd1234
      MYSQL_DATABASE: mydb
    volumes:
      - ./mysql_data:/var/lib/mysql
    ports:
      - 3307:3306
    healthcheck:
      # ping 명령어로 5초 간격 최대 10번 시도
      test: ["CMD", "mysqladmin", "ping"]
      interval: 5s
      retries: 10

  my-cache-server:
    image: redis
    ports:
      - 6379:6379
    healthcheck:
      # ping 명령어로 5초 간격 최대 10번 시도
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      retries: 10

```

<br>

```properties
spring.application.name=demo

# localhost로 하면 연결이 실패하고, Docker를 띄울 때 설정한 서비스 이름으로 지정해야한다.
spring.datasource.url:jdbc:mysql://my-db:3306/mydb
spring.datasource.username:root
spring.datasource.password:pwd1234
spring.datasource.driver-class-name:com.mysql.cj.jdbc.Driver

# localhost로 하면 연결이 실패하고, Docker를 띄울 때 설정한 서비스 이름으로 지정해야한다.
spring.data.redis.host:my-cache-server
spring.data.redis.port:6379
```