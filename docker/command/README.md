# 🧑🏻‍💻 Docker Command

---

> [!TIP]
> 아래 명령어 자리에 container_id 자리에 container_name이 들어가도 대부분 동일하게 동작한다.  
> 역도 성립

<br>

### ✅ Docker Image 명령어

```shell
# docker image 목록
docker image ls
```

<br>

```shell
# 최신 버전의 이미지 다운로드
docker pull nginx

# 위와 같은 명령어다.
docker pull nginx:latest

# 특정 버전(태그)의 이미지 다운로드(docker hub에서 찾아올 수 있다)
docker pull nginx:1.30
```

<br>

```shell
# 특정 ID의 이미지 삭제 - container에서 미사용 중인 이미지만 삭제 가능
docker image rm {image_id}
docker image rm {image_name}

# 중단된 컨테이너에서 사용 중인 이미지더라도 강제로 삭제 - 실행 중인 컨테이너에서 사용 중이면 삭제 X
docker image rm -f {image_id}
docker image rm -f {image_name}

# 컨테이너에서 사용 중이지 않은 전체 이미지들 삭제
docker image rm $(docker images -q)
```

<br>

### ✅ Docker Container 명령어

```shell
# 실행 중인 컨테이너 목록
docker ps

# 중단된 컨테이너까지 포함한 목록
docker ps -a
```

<br>

```shell
# nginx 이미지를 기반으로 컨테이너 생성
docker create nginx
```

<br>

```shell
# 특정 컨테이너 실행
docker start {container_id}
```

<br>

```shell
# docker create + start 같이 데몬으로 실행
docker run -d nginx 

# docker name도 부여하여 데몬으로 실행
docker run -d --name my-web-server nginx

# host 포트 4000번과 컨테이너 포트 80으로 연결하여 실행
docker run -d -p 4000:80 nginx
```

![docker_port_mapping.png](../res/docker_port_mapping.png)

<br>

```shell
# 컨테이너 정상 종료
docker stop {container_id}

# 컨테이너 강제 종료
docker kill {container_id}
```

<br>

```shell
# 중지된 컨테이너 삭제
docker rm {container_id}
```


<br>

```shell
# 중지된 컨테이너 전체 삭제 (볼륨은 건드리지 않는다.)
docker container prune

# 중지된 컨테이너 전체 삭제 (볼륨은 건드리지 않는다.)
docker rm $(docker ps -qa)
```


<br>

```shell
# 컨테이너 실행
# -i: --interactive / 표준 입력(STDIN)을 열어둔 채로 유지한다.
# -t: --tty / 가상 터미널 할당. 터미널처럼 보이는 환경을 만들어준다. 
docker exec -it {container_name} bash
```

<br>

```shell
# 전체 로그 확인
docker logs {container_name}

# 끝에 10줄만 로그 확인
docker logs --tail 10 {container_name}

# docker 로그 실시간 확인
docker logs -f {container_name}

# 기존 로그는 안 보고 새로운 로그부터만 조회하겠다
docker logs --tail 0 -f {container_name}
```

<br>

### ✅ Docker Compose 명령어

```shell
# docker-compose.yml 백그라운드로 실행. 로그를 확인하면서 실행하고 싶다면 -d 옵션 생략
docker compose up -d

# 특정 컨테이너만 실행
docker-compose up -d {container_name}
```

<br>

```shell
# docker-compose 종료
docker compose down
```

<br>

```shell
# docker 재실행
docker compose restart {container_name}
```

