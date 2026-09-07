# 🧑🏻‍💻 리눅스 명령어 기록

---

- [✅ 파일/권한 기본기](#-파일권한-기본기)
- [✅ 프로세스 및 리소스 확인](#-프로세스-및-리소스-확인)
- [✅ 로그 확인 및 텍스트 처리](#-로그-확인-및-텍스트-처리)
- [✅ 네트워크 상태 점검](#-네트워크-상태-점검)
- [✅ 디스크 및 시스템 리소스 점검](#-디스크-및-시스템-리소스-점검)
- [✅ Java Application 장애 진단 도구](#-java-application-장애-진단-도구)


## ✅ 파일/권한 기본기

### 🚀 `ls` - 디렉토리 내용 확인
```shell
ls -al
```
> [!NOTE]
> - `l`: long format. 권한, 소유자, 크기, 수정일자까지 한 줄로 보여준다.
> - `a`: all. `.`으로 시작하는 숨김 파일까지 모두 보여준다.
> - `h`: human-readable. `-lh`로 조합하면 용량을 K/M/G 단위로 보기 좋게 표시한다.
> - 결과 예시: `-rw-r--r--@  1 kyeongchanwoo  staff   3.1K May 12  2024 .gitignore`

<br>

> [!TIP]
> 권한표기 `-rw-r--r--@` 읽기
> - 맨 앞 1자리: 파일 종류에 해당
>   - `-`: 일반 파일
>   - `d`: 디렉토리
>   - `l`: 심볼릭 링크
> - 그 다음 9자리를 3자리씩 끊어서: `rwx`(소유자)/`r--`(그룹)/`r--`(기타 사용자)
>   - `r`: read
>   - `w`: write
>   - `x`: execute

<br>

### 🚀 `find` - 파일 탐색(장애 시점 추적)
```shell
find /home/was01/logs -name "*.log" -mtime -1
find /home/was01/logs -newer /tmp/marker -type f
```
> [!NOTE]
> - `-name`: 파일명 패턴 검색. 와일드카드 사용 가능.
> - `-mtime -1`: 최근 1일 이내에 수정된 파일.
>   - `mtime +7`이면 7일보다 오랜된 파일
> - `-newer 기준파일`: 기준파일보다 최근에 수정된 파일만 검색.
>   - "장애 발생 시각 이후 바뀐 파일이 뭐지?"할 때 사용하면 유용
> - `-type f` / `-type d`: 파일만, 또는 디렉토리만 검색

<br>

### 🚀 `chmod` - 권한 변경
```shell
chmod 755 start.sh
chmod +x deploy.sh
```
> [!NOTE]
> - 숫자 방식: `rwx`를 각각 4, 2, 1로 보고 합산.
> - `+x`: 소유자, 그룹, 기타 사용자 모두에게 실행권한만 추가

<br>

> [!WARNING]
> - "필요한 최소 권한만 준다"는 기준으로, 실행 파일은 `750`이나 `755`, 설정 파일이나 비밀번호가 담긴 파일은 `600`처럼 범위를 좁혀서 부여한다. 
> - 특히 `x`까지 전체 공개되면, 악의적인 사용자나 침해당한 다른 계정이 그 파일을 스크립트처럼 실행시켜 임의 명령을 수행할 여지가 생긴다.

<br>

### 🚀 기타 필수
```shell
mkdir -p /home/was01/backup/2026
cp -r src/ dest/
mv old.log old.log.bak
rm -rf temp/
```
> [!NOTE]
> - `mkdir -p`: 중간 경로가 없어도 한 번에 생성
> - `cp -r`: 디렉토리 통째로 복사
>   - `dest`가 존재하지 않는 경우: `dest` 자체가 `src`의 사본이 된다.
>   - `dest`가 이미 존재하는 경우: `src` 전체가 `dest` 안으로 들어간다.
>   - `dest/src`가 이미 존재하는 경우: 병합이 된다. 
>     - 이름이 같은 파일을 덮어쓰고, `src`에 없는 파일은 기존 그대로 유지
> - `mv`: 기존에 `old.log.bak`이 존재하면 덮어쓰고, 없으면 단순히 이름 변경
> - `rm -rf`: 강제 삭제. **조심히 써야함.**

<br>

> [!IMPORTANT]
> `cp`나 `mv`는 `-i`를 쓰는 것을 습관화하자.  
> 덮어쓰기 직전에 overwrite? (y/n) 프롬프트가 뜬다.
```shell
cp -ri src/ dest/
mv -i old.log old.log.bak
```

<br>

## ✅ 프로세스 및 리소스 확인

### 🚀 `ps` - 현재 실행 중인 프로세스 목록
```shell
ps -ef | grep java
```
> [!NOTE]
> - `-e`: every process. 모든 사용자의 프로세스 표시
> - `-f`: full format. PPID(부모 프로세스), 시작 시각, 실행 명령어까지 상세히 표시.
> - `| grep java`: 파이프로 결과를 넘겨서 "java"가 포함된 줄만 필터링.
>   - WAS 프로세스의 PID를 찾을 때 가장 많이 쓰는 조합이다.

<br>

### 🚀 `top` / `htop` - 실시간 리소스 모니터링
```shell
top
htop
```
> [!NOTE]
> - `top`은 기본 내장, `htop`은 별도 설치가 필요하지만 색상과 조작이 편하다.
> - 화면에서 `%CPU`, `MEM` 컬럼을 보고 어떤 프로세스가 리소스를 많이 먹는지 확인할 수 있다.
> - `top` 실행 중 `Shift + M`➡ 메모리 사용량 순 정렬
> - `top` 실행 중 `Shift + P`➡ CPU 사용량 순 정렬
> - `q`: 종🚀

<br>

### 🚀 `kill` - 프로세스 종료
```shell
kill -15 12345
kill -9 12345
```
> [!NOTE]
> - `-15`(SIGTERM): "정상적으로 종료해달라"는 신호다.
>   - 프로세스가 자원을 정리하고 종료할 기회를 준다.
> - `-9`(SIGKILL): 강제 종료다.
>   - 프로세스가 응답이 없을 때 최후의 수단으로 사용한다.
>   - 트랜잭션 중간에 강제 종료하면 데이터 정합성 문제가 생길 수 있기 때문에 신중해야 한다.


<br>

### 🚀 `nohup`, `&` - 백그라운드 실행
```shell
nohup java -jar app.jar > app.log 2>&1 &
```
> [!NOTE]
> - `nohup`: no hang up. 터미널 세션이 끊겨도(SSH 접속 종료 등) 프로세스가 계속 실행되도록 유지
> - `&`: 명령어를 백그라운드로 실행. ➡ 터미널을 점유하지 않고 다른 작업 가능
> - `>`: 표준 출력을 `app.log`에 저장
> - `2>&1`: 표준 에러(2번)를 표준 출력(1번)과 합쳐서 같은 파일에 기록.
>   - `0`: 표준 입력(stdin)
>   - `1`: 표준 출력(stdout)
>   - `2`: 표준 에러(stderr)
>   - `2>&1`: fd 1이 가리키는 곳을 fd 2가 가리키게 해라

<br>

> [!TIP]
> 참고로, 이건 여러 명령어가 아니라 명령어 1개 + 옵션들 형태다.  
> `nohup [실행할 명령어]`  
> 
> 만약 리눅스에서 한 줄에 여러 명렁어를 쓰고 싶다면
> - `;`: 순차 실행
> - `&&`: 성공 시 다음 실행
> - `|`: 파이프

| 기호  | 이름   | 동작                                       |
|-----|------|------------------------------------------|
| `;` | 세미콜론 | 앞 명령의 성공/실패 여부와 상관없이, 무조건 순서대로 다음 명령을 실행 |
| `&&` | AND  | 앞 명령이 성공(종료코드 0)했을 때만 다음 명령을 실행          |
| `ㅣㅣ` | OR   | 앞 명령이 실패(종료코드 0이 아님)했을 대만 다음 명령을 실행      |
| `ㅣ`  | 파이프  | 앞 명령의 출력을 다음 명령의 입력으로 그대로 연결             |

<br>

## ✅ 로그 확인 및 텍스트 처리

### 🚀 `tail` - 파일 끝부분 확인(실시간 모니터링)
```shell
tail -f app.log
tail -n 200 app.log
```
> [!NOTE]
> - `-f`: follow. 파일에 새로운 로그가 쌇이는 걸 실시간으로 화면에 보여줌.
> - `-n 200`: 마지막 200줄만 출력

<br>

### 🚀 `head` -  파일 앞부부 확인
```shell
head -n 50 app.log
```
> [!NOTE]
> 로그 파일이 언제부터 기록되기 시작했는지, 헤더 정보를 볼 때 사용.

<br>

### 🚀 `grep` -  패턴 검색
```shell
grep "ERROR" app.log
grep -i "exception" app.log
grep -n "OutOfMemory" app.log
grep -c "ERROR" app.log
grep -A 5 -B 2 "NullPointerException" app.log
```
> [!NOTE]
> global regular expression print
> - `-i`: 대소문자 구분 안 함(ignore case)
> - `-n`: 매칭된 줄의 줄 번호까지 표시
> - `-c`: 매칭된 줄의 개수만 카운트. "오늘 에러가 몇 건 났지?" 확인용
> - `-A 5`: 매칭된 줄 이후(After) 5줄까지 함께 출력
> - `-B 5`: 매칭된 줄 이전(Before) 2wnf cnffur

<br>

### 🚀 `awk` -  컬럼 단위 텍스트 처리
```shell
awk '{print $1 $4}' access.log
awk -F',' '{print $3}' data.csv
```
> [!NOTE]
> - 기본적으로 공백을 구분자로 컬럼을 나눈다. `$1`은 첫 번째 컬럼, `$4`는 네 번째 컬럼.
> - `-F','`: 구분자를 콤마로 지정. CSV 형태의 로그나 데이터 파싱에 사용.

<br>

### 🚀 `sed` - 텍스트 치환
```shell
sed 's/ERROR/WARN/g' app.log
sed -n '10,20p' app.log
```
> [!NOTE]
> - `s/찾을 것/바꿀 것/g`: 전체(global) 치환. `g`가 없으면 각 줄에서 첫 번째만 치환됨.
> - `-n '10,20p'`: 10번째부터 20번째 줄까지만 출럭(print). `tail`/`head`로 안 되는 중간 범위를 볼 때 유용.

<br>

### 🚀 `less` - 대용량 파일 보기
```shell
less app.log
```
> [!NOTE]
> - `cat`과 달리 파일 전체를 메모리에 올리지 않고 필요한 부분만 읽어서, 몇 GB짜리 로그 파일도 가볍게 열 수 있다.
> - `/검색어` + Enter: 검색.
> - `n`: 다음 매칭으로 이동
> - `q`: 종료

<br>

### 🚀 `wc` - 개수 세기(Word Count)
```shell
wc -l app.log
```
> [!NOTE]
> - `-l`: 줄의 개수. 로그 총 건수 파악할 때

<br>

## ✅ 네트워크 상태 점검

### 🚀 `netstat`/`ss` - 포트 및 연결 상태(network statistic/socket statistic)
```shell
netstat -an | grep 8080
ss -tnlp
```
> [!NOTE]
> - `-a`: 모든 연결 표시
> - `-n`: 호스트명 대신 IP/포트 번호로 표시
> - `ss`는 `netstat`의 최신 명령어(더 빠름)
>   - `-t`: TCP만
>   - `-n`: 숫자로 표시
>   - `-l`: listening 상태만.
>   - `-p`: 

<br>

### 🚀 `ping` - 네트워크 연결 확인
```shell
ping -c 4 10.0.0.5
```
> [!NOTE]
> `-c 4`: 4번만 보내고 종료.

<br>

### 🚀 `curl` - HTTP 요청 테스트(client URL / see URL 말장난)
```shell
curl -I http://localhost:8080/health
curl -v http://localhost:8080/api/test
```
> [!NOTE]
> - `-I`: 헤더만 요청(HEAD 요청). 응답 코드만 빠르게 확인할 때
> - `-v`: verbose. 요청/응답 전체 과정을 상세히 출력. API 연동 장애 디버깅에 필수

<br>

### 🚀 `traceroute` - 경로 추적(trace + route)
```shell
traceroute 10.0.0.5
```
> [!NOTE]
> 목적지까지 거치는 네트워크 구간을 순서대로 보여줘서, 어느 구간에서 지연/차단이 생기는지 파악할 때 사용

<br>

## ✅ 디스크 및 시스템 리소스 점검

### 🚀 `df` - 디스크 사용량(disk free)
```shell
df -h
```
> [!NOTE]
> - `-h`: human-readable. GB/MB 단위로 표시.
> - `Use%`가 90% 이상이면 위험 신호.

<br>

### 🚀 `du` - 디렉터리/파일 용량(disk usage)
```shell
du -sh /home/was01/logs/*
```
> [!NOTE]
> - `-s`: summary. 하위 디렉토리 용량을 합산해서 한 줄로.
> - `-h`: human-readable
> - 어떤 디렉토리가 용량을 많이 차지하는지 찾을 때 사용

<br>

### 🚀 `free` - 메모리 상태
```shell
free -h
```
> [!NOTE]
> - `total`, `used`, `free`, `available` 컬럼 확인.
> - `available`이 실제로 새 프로세스가 쓸 수 있는 여유 메모리다.

<br>

### 🚀 `vmstat`(virtual memory statistics) - 시스템 전반 상태(CPU/메모리/스왑)
```shell
vmstat 2 5
```
> [!NOTE]
> - 2초 간격으로 5번 반복 출력
> - `si`, `so`(swap in/out) 값이 0이 아니라면 메모리 부족으로 스왑이 발생 중이라는 뜻 - 성능 저하의 주요 원인

<br>

### 🚀 `uptime` - 시스템 부하 확인
```shell
uptime
```
> [!NOTE]
> - load average(1분/5분/15분 평균 부하)를 보여줌.
> - CPU 코어 수 대비 이 값이 지속적으로 높으면 부하 과다 상태다.

<br>

## ✅ Java Application 장애 진단 도구

### 🚀 `jps`(Java Process Status) - JVM 프로세스 목록
```shell
jps -l
```
> [!NOTE]
> - `-l`: 클래스 전체 경로 또는 jar 경로까지 표시.
> - `ps -ef | grep java`보다 Java 프로세스만 깔끔하게 확인 가능

<br>

### 🚀 `jstack`(Java Stack, 스레드의 콜 스택 덤프) - 스레드 덤프(데드락/행 진단)
```shell
jstack 12345 > threaddump.txt
```
> [!NOTE]
> - 특정 시점의 모든 스레드 상태를 스냅샷으로 뜸.
> - 결과에서 `BLOCKED`, `WAITING` 상태의 스레드와, `Found one Java-Level deadlock` 메시지 유무를 확인.
> - 응답이 멈춘(hang) 상황에서 원인 파악에 가장 결정적인 도구다.

<br>

### 🚀 `jstat`(Java statistics monitoring tool) - GC 및 메모리 상태 모니터링
```shell
jstat -gcutil 12345 1000 10
```
> [!NOTE]
> - `-gcutil`: GC 영역별 사용률(%)을 보여줌.
> - `1000 10`: 1000ms(1초) 간격으로 10번 출력
> - Full GC가 너무 자주 발생하는지, Old 영역이 계속 차오르는지(메모리 누수 의심) 확인할 때 사용

<br>

### 🚀 `jmap`(Java memory map) - 힙 덤프(메모리 누수 분석)
```shell
jmap -dump:format=b,file=heapdump.hprof 12345
```
> [!NOTE]
> - 힙 메모리 전체를 파일로 떠서, 이후 Eclipse MAT 같은 툴로 분석
> - `-dump:format=b,file=heapdump.hprof`: `-dump` 옵션에 콤마로 세부 설정 2가지를 붙인 것이다.
>   - `format=b`: 덤프 파일의 형식 지정
>     - `b`는 binary의 약자로, Eclipse MAT 같은 분석 도구가 읽을 수 있는  이진 형식으로 저장하라는 뜻이다.
>     - 현재 jmap에서는 사실상 `b`가 유일하게 쓰이는 표준 포맷이다.
>   - `file=heapdump.hprof`: 결과를 저장할 파일 경로와 이름 지정
>     - `.hprof`는 관례적으로 붙이는 확장자(Heap Profile)로, Eclipse MAT 등 분석 도구들이 이 확장자를 보고 힙 덤프 파일임을 인식한다.

> [!WARNING]
> OutOfMemoryError 원인 분석 시 사용하지만, 덤프 자체가 무겁고 서비스에 영향을 줄 수 있어 운영 환경에서는 신중하게 사용해야 한다.











