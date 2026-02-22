# 🧑🏻‍💻 Docker Command
```shell
# docker-compose.yml 백그라운드로 실행. 로그를 확인하면서 실행하고 싶다면 -d 옵션 생략
docker compose up -d

# 특정 컨테이너만 실행
docker-compose up -d {container_name}
```

```shell
# 컨테이너 실행
docker exec -it {container_name} bash
```

```shell
# docker-compose 종료
docker compose down
```

```shell
# docker 재실행
docker compose restart {container_name}
```

```yaml
# docker 로그 실시간 확인
docker logs -f {container_name}
```