# 💻 레디스 시작하기
> linux 환경에서 진행하였습니다. 

## ✅ 레디스 설치하기

```shell
-- 최신 버전 다운로드
wget https://download.redis.io/redis-stable.tar.gz
```
레디스를 빌드하기 위해서는 gcc 버전 4.6 이상이 필요하므로 gcc를 미리 설치하는 것이 좋다.
-  **GNU Compiler Collection**의 약자로, GNU 프로젝트의 일환으로 개발된 무료 오픈 소스 컴파일러 모음이다.

```shell
-- gcc 설치
yum install -y gcc

-- 압축 파일 해제 후 빌드
tar -xzvf redis-stable.tar.gz
mv redis-stable redis
cd redis
make
```

> `cc: not found` 에러가 발생하는 경우가 있다.
> 이럴 때는 C 컴파일러가 설치되어있지 않아 발생한 것이므로, 하기와 같이 패키지를 설치한다.
> ```shell
> $ sudo apt update
> $ sudo apt install build-essential pkg-config
> ```

<br>

기본 디렉터리 내의 bin 디렉터리에 실행 파일을 복사하기 위해 `make install` 커맨드를 프리필스 지정화 함께 수행하였다.
```shell
# 프리픽스 경로: /home/{계정 username}/redis
$ make PREFIX=/home/redis/redis install
```

<br>

레디스 포르가운드(foreground) 모드(사용자가 현재 기기 화면에서 직접 보고 상호작용하는 상태)로 실행하기
```shell
$ bin/redis-server redis.conf
```
![redis_foreground_mode.png](../res/redis_foreground_mode.png)


<br>

## ✅ 레디스 환경 구성
### 💡 Open files 확인
> 레디스의 기본 maxclients 설정 값은 10,000이다.  
이는 레디스 프로세스에서 받아들일 수 있는 최대 클라이언트의 개수를 의미한다.  
하지만 이 값은 레디스를 실행하는 서버의 파일 디스크립터 수에 영향을 받는다.  
레디스 프로세스 내부적으로 사용하기 위해 예약한 파일 디스크립터 수는 32개로, maxclients 값에 32를 더한 값보다 서버의 최대 파일 디스크립터 수가 작으면 레디스는 실행될 때 자동으로 그 수에 맞게 조정된다. 

```shell
# 현재 서버의 파일 디스크립터 수 확인
$ ulimit -a | grep open
```

open files의 값이 10,032보다 작다면 `/etc/security/limits.conf` 파일에 다음과 같은 구문을 추가한다.
```text
*               hard    nofile          100000
*               soft    nofile          100000
```

서버 재접속 후 다시 파일 디스크립터 수 확인해보면 설정한 값이 반영돼있는 것을 확인할 수 있다.
```shell
$ ulimit -a | grep open
open files                          (-n) 100000
```

<br>

**참고 자료**  
[개발자를 위한 레디스](https://product.kyobobook.co.kr/detail/S000210785682)