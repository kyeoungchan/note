# 🧑🏻‍💻 레디스 데이터 백업 방법

- [레디스에서 데이터를 영구 저장하기](#-레디스에서-데이터를-영구-저장하기)
- [RDB 방식의 데이터 백업](#-rdb-방식의-데이터-백업)
  - [특정 조건에 자동으로 RDB 파일 생성](#-특정-조건에-자동으로-rdb-파일-생성)
  - [수동으로 RDB 파일 생성](#-수동으로-rdb-파일-생성)
  - [복제를 사용할 경우 자동으로 RDB 파일 생성](#-복제를-사용할-경우-자동으로-rdb-파일-생성)
- [AOF 방식의 데이터 백업](#-aof-방식의-데이터-백업)
  - [AOF 파일을 재구성하는 방법](#-aof-파일을-재구성하는-방법)
  - [자동 AOF 재구성](#-자동-aof-재구성)
  - [AOF 타임스탬프](#-aof-타임스탬프)
  - [AOF 파일 복원](#-aof-파일-복원)
- [백업을 사용할 때 주의할 점](#-백업을-사용할-때-주의할-점)

## ❗️ 레디스에서 데이터를 영구 저장하기
> 레디스에서 모든 데이터는 메모리에서 관리된다.  
> ➡ 레디스 인스턴스 혹은 레디스가 실행되는 서버의 장애로 인해 레디스 인스턴스가 재시작될 경우 메모리에 상주해 있던 레디스의 모든 데이터는 손실 위험이 있다.

> 레디스를 복제 구조로 사용할 경우, 복제본이 있으니 데이터 백업의 필요성을 못 느낄 수 있다.  
> 하지만 백업과 복제는 목적부터가 다르다.  
> - 복제: 가용성을 위함
> - 백업: 장애 상황에서 데이터의 복구
> 
> 만약 개발자의 실수나 프로그램의 버그로 인해 마스터 노드에서 의도치 않은 데이터 손상이 일어날 경우, 이는 바로 복제본으로 전달된다.  
> ➡ 복제 구조만으로는 데이터를 안전하게 유지 ❌

데이터를 안전하게 저장하기 위해 레디스에서는 `RDB`와 `AOF` 2가지의 백업 방식을 지원한다.
1. `AOF`(Append Only File): 레디스 인스턴스가 처리한 모든 쓰기 작업을 차례대로 기록한다.  
   ➡ 복원 시에는 파일을 다시 읽어가며 데이터 세트 구성
2. `RDB`(Redis Database): 일정 시점에 메모리에 저장된 데이터 전체를 저장(snapshot 방식)

<br>

```redis
127.0.0.1:6379> SET key1 a
OK
127.0.0.1:6379> SET key1 apple
OK
127.0.0.1:6379> SET key2 b
OK
127.0.0.1:6379> DEL key2
1
```
위 커맨드가 실행됐을 경우, `AOF`와 `RDB` 파일에는 데이터가 아래와 같이 저장된다.
```text
# AOF
set key1 a
set key1 apple
set key2 b
del key2
```
```text
# RDB
key1 -> apple
```

> 참고로, 실제로 둘 다 우리가 읽을 수 없는 형태의 파일이다.  
> - `RDB`: 바이너리 형태
> - `AOF`: 레디스 프로토콜(RESP) 형태

각각 장단점이 있다.  

|구분| `RDB`                                             |`AOF`|
|:---:|---------------------------------------------------|---|
|장점| 시점 단위로 여러 백업본을 저장할 수 있다. <br> `AOF` 파일보다 복원이 빠르다. |원하는 시점으로 복구할 수 있다.|
|단점|특정 시점으로의 복구는 불가능하다.|크기가 크고, 주기적으로 압축해 재작성해야 한다.|

하나의 인스턴스에서 `RDB`와 `AOF` 옵션을 동시에 사용하는 것도 가능하며, 일반적인 `RDB` 만큼의 데이터 안정성을 원하는 경우, 2가지 백업 방식을 동시에 사용하기를 권장한다.

<br>

레디스에서 데이터를 복원할 수 있는 시점은 서버가 재시작될 때뿐이며, 레디스 인스턴스의 실행 도중에 데이터 파일을 읽어올 수 있는 방법은 없다.  
레디스 서버는 재시작될 때 `AOF` 파일이나 `RDB` 파일이 존재하는지 확인한 뒤, 파일이 있을 때에는 파일을 로드한다.  
레디스는 `RDB` 파일보다 `AOF` 파일이 더 내구성이 보장된다고 판단하기 때문에, 2개 다 있으면 `AOF`의 데이터를 로드한다.

<br>

## ❗️ RDB 방식의 데이터 백업
- [특정 조건에 자동으로 RDB 파일 생성](#-특정-조건에-자동으로-rdb-파일-생성)
- [수동으로 RDB 파일 생성](#-수동으로-rdb-파일-생성)
- [복제를 사용할 경우 자동으로 RDB 파일 생성](#-복제를-사용할-경우-자동으로-rdb-파일-생성)

RDB 파일은 레디스에서 데이터를 백업하기 위한 가장 단순한 방법이다.  
예를 들어, 한 시간에 한 번씩 RDB 파일을 생성할 수 있다.  
RDB 파일이 저장될 때마다 원격 저장소로 파일을 옮겨 2차 백업을 수행한다면 데이터 센터 장애 등 더 큰 장애에도 대처할 수 있다.

<br>

그러나 사용자가 지정한 시간 단위로 파일이 저장되기 때문에, 저장 시점부터 장애가 발생한 직전까지의 데이터는 손실될 수 있다는 리스크가 있다.

### ✅ 특정 조건에 자동으로 RDB 파일 생성
```text
save <기간(초)> <기간 내 변경된 키의 개수>
dbfilename <RDB 파일 이름>
dir <RDB 파일이 저장될 경로>
```

레디스의 설정 파일에서 `save` 옵션을 사용해 원하는 조건에 RDB 파일을 저장하도록 설정할 수 있다.  
- `save`: 일정한 기간(초) 동안 변경된 키의 개수가 조건에 맞을 때 레디스 서버는 자동으로 RDB 파일을 저장한다.
- `dbfilename`: RDB 파일 이름이 해당 옵션값으로 지정되고, 기본 값은 `dump.rdb`이다.
- `dir`: 파일이 저장될 경로다.

```text
# redis.conf

# 900초(15분) 동안 1개 이상의 키가 변경된 경우 RDB 파일 생성
save 900 1

# 300초(5분) 동안 10개 이상의 키가 변경된 경우 RDB 파일 생성
save 300 10

# 60초(1분) 동안 10,000개 이상의 키가 변경된 경우 RDB 파일 생성
save 60 10000
```

> 이미 레디스 인스턴스가 실행 중인 상태에서 RDB 저장을 비활성화하고 싶다면 `CONFIG SET` 커맨드를 통해 `save` 파라미터를 `""`로 초기화시키면 된다.

```redis
# 현재 적용된 save 옵션 확인
127.0.0.1:6379> CONFIG GET save
save
3600 1 300 100 60 10000

# save 옵션 초기화
127.0.0.1:6379> CONFIG SET save ""
OK

# 현재 적용된 save 옵션 확인
127.0.0.1:6379> CONFIG GET save
save

# redis.conf 파일 재작성
127.0.0.1:6379> CONFIG REWRITE
OK
```

> `CONFIG REWRITE`  
> 레디스 인스턴스가 실행 중인 상태에서 설정 파일을 변경하는 것은 실행 중인 레디스 인스턴스에는 반영되지 않는다.  
> 실행 중인 레디스 인스턴스에서 파라미터를 수정할 때에는 `redis-cli`에서 직접 `CONFIG SET` 커맨드로 설정을 변경한 뒤, `CONFIG REWRITE` 커맨드를 이용해 설정 파일을 재작성하는 과정을 거쳐야 한다.  

<br>

### ✅ 수동으로 RDB 파일 생성
`SAVE`, `BGSAVE` 커맨드를 이용하면 원하는 시점에 직접 RDB 파일을 생성할 수 있다.  
- `SAVE`
  - 동기 방식으로 파일을 저장한다.
  - 파일 생성이 완료될 때까지 다른 모든 클라이언트의 명령을 차단한다.
  - 메모리 전체를 스캔해 파일이 저장되기까지 1분이 걸린다면, 그 1분 동안 레디스 인스턴스에 연결된 다른 클라이언트는 아무런 명령도 수행할 수 없다.
  - 🚫 따라서 일반적인 운영 환경에서는 `SAVE` 커맨드를 되도록 사용하지 않는 것이 좋다.
- `BGSAVE`
  - `fork`를 호출해 자식 프로세스를 생성하며, 생성된 자식 프로세스가 백그라운드에서 RDB 파일을 생성한 뒤 종료된다.
  - 레디스를 이용하는 다른 클라이언트는 원래대로 부모 프로세스를 이용해서 처리되기 때문에 파일 저장에는 영향을 받지 않는다.

> `BGSAVE`를 통해 백그라운드로 데이터가 저장되고 있을 때 `BGSAVE` 커맨드를 또 호출하면 에러를 반환한다.  
> 이런 상황에서는 `BGSAVE`와 함께 `SCHEDULE` 옵션을 사용할 수 있는데, 이미 파일이 백그라운드에서 저장 중일 때 이 커맨드를 입력받은 레디스는 일단 OK를 반환한 뒤, 기존에 진행 중이던 백업이 완료됐을 때 다시 `BGSAVE`를 실행한다.

```redis
127.0.0.1:6379> bgsave
Background saving started

# RDB 파일이 정상적으로 저장됐는지 확인
# 마지막으로 RDB 파일이 저장된 시점을 유닉스 타임스탬프로 반환한다.
127.0.0.1:6379> LASTSAVE
1766009952

# 같다.
127.0.0.1:6379> LASTSAVE
1766009952

# 수동으로 저장
127.0.0.1:6379> BGSAVE
Background saving started

# 달라졌다.
127.0.0.1:6379> LASTSAVE
1766010744
```

<br>

### ✅ 복제를 사용할 경우 자동으로 RDB 파일 생성
복제본에서 `REPLICAOF` 커맨드를 이용해 복제를 요청하면 마스터 노드에서는 RDB 파일을 새로 생성해 복제본에 전달한다.  
혹은 이미 복제 연결이 돼 있는 상태에서 네트워크 등의 이슈로 복제가 끊어졌다가 복구된 경우 복제 재 연결이 발생하면서 마스터 노드에서 복제본으로 RDB 파일이 전송된다.  

<br>

## ❗️ AOF 방식의 데이터 백업
- [AOF 파일을 재구성하는 방법](#-aof-파일을-재구성하는-방법)
- [자동 AOF 재구성](#-자동-aof-재구성)
- [수동 AOF 재구성](#-수동-aof-재구성)
- [AOF 타임스탬프](#-aof-타임스탬프)
- [AOF 파일 복원](#-aof-파일-복원)

AOF는 레디스 인스턴스에서 수행된 모든 쓰기 작업의 로그를 차례로 기록한다.  
➡ 실수로 `FLUSHALL` 커맨드로 데이터를 모두 날려버렸다 해도, AOF 파일을 직접 열어 `FLUSHALL` 커맨드만 삭제한 뒤, 레디스를 재시작시킨다면 커맨드를 실행하기 직전까지로 데이터를 바로 복구할 수 있다.  

<br>

```text
appendonly yes
appendfilename "appendonly.aof"
appenddirname "appendonlydir"
```
- `appendonly`: 설정 파일에서 `appendonly` 옵션을 `yes`로 지정하면 AOF 파일에 주기적으로 데이터가 저장된다.  
- `appendfilename`: AOF 파일은 `appendfilename` 옵션에 설정한 이름으로 생성된다.
  - 디폴트는 `appendonly.aof`다.
- `appenddirname`
  - 경로가 아닌 디렉토리 이름만 지정할 수 있으며, 해당 디렉토리 하위에 저장된다.
  - `dir` 옵션 하위에 생성된다.
  - 확인해보니 `dir "/home/redis/redis"` 이렇게 되어있다.

```redis
# 레디스 인스턴스가 실행되는 동안 설정 파일 수정
127.0.0.1:6379> CONFIG GET appendonly
appendonly
no
127.0.0.1:6379> CONFIG SET appendonly yes
OK
127.0.0.1:6379> CONFIG GET appendonly
appendonly
yes
127.0.0.1:6379> CONFIG REWRITE
OK
```

<br>

```redis
# 레디스 명령어 수행 결과

127.0.0.1:6379> SET key1 apple
OK
127.0.0.1:6379> SET key1 beer
OK
127.0.0.1:6379> DEL key1
1
127.0.0.1:6379> DEL no_existing_key
0
```

```shell
# appendonly.aof 파일 찾기

redis@redisvm:~/redis/appendonlydir$ cd /home/redis/redis/appendonlydir

# appendonly.aof에 일련 번호가 추가돼서 저장되는 거 같다.
redis@redisvm:~/redis/appendonlydir$ ll
total 20
drwxr-xr-x  2 redis redis 4096 Dec 17 22:51 ./
drwxrwxr-x 11 redis redis 4096 Dec 17 22:48 ../
-rw-rw-r--  1 redis redis 2946 Dec 17 22:46 appendonly.aof.1.base.rdb
-rw-r--r--  1 redis redis  113 Dec 17 22:47 appendonly.aof.1.incr.aof
-rw-r--r--  1 redis redis  102 Dec 17 22:46 appendonly.aof.manifest
```
```aof
// appendonly.aof.1.incr.aof 내용
*3
$3
SET
$4
key1
$5
apple
*3
$3
SET
$4
key1
$4
beer
*2
$3
DEL
$4
key1
```

> `DEL no_existing_key` 커맨드는 메모리가 수정되는 작업이 아니다.  
> AOF 파일은 메모리상의 데이터가 변경되는 커맨드만 기록되기 때문에 해당 커맨드 작업은 기록하지 않는다.


<br>

하지만 AOF 파일은 사용자가 실행한 커맨드를 그대로 저장하지는 않는다.  
예를 들어 list에서 블로킹 기능을 지원하는 `BRPOP` 커맨드는 AOF에서 굳이 블로킹 기능을 명시할 필요는 없기 때문에 `RPOP`으로 기록된다.
```redis
127.0.0.1:6379> RPUSH mylist a b c d e
(integer) 9
127.0.0.1:6379> BRPOP mylist 1
1) "mylist"
2) "e"
```
```text
*7
$5
RPUSH
$6
mylist
$1
a
$1
b
$1
c
$1
d
$1
e
*2
$4
RPOP
$6
mylist
```

<br>

레디스가 실행되는 아키텍처에 따라 부동소수점을 처리하는 방식이 다를 수 있기 때문에 AOF 파일에는 증분 후의 값을 직접 SET하는 커맨드로 변경돼 저장된다.
```redis
127.0.0.1:6379> SET counter 100
OK
127.0.0.1:6379> INCRBYFLOAT counter 50
"150"
```
```text
*3
$3
SET
$7
counter
$3
100
*4
$3
SET
$7
counter
$3
150
$7
KEEPTTL
```

<br>

### ✅ AOF 파일을 재구성하는 방법
- [레디스 버전 7 이전의 AOF](#-레디스-버전-7-이전의-aof)
- [레디스 버전 7 이후의 AOF](#-레디스-버전-7-이후의-aof)

AOF 파일을 이용한 백업 기능을 안정적으로 사용하려면 점점 커지는 파일을 주기적으로 압축시키는 재구성(rewrite) 작업이 필요하다.  
RDB에서와 마찬가지로 특정 조건에 자동으로 재구성되도록 설정할 수도 있으며, 사용자가 원하는 시점에 커맨드를 이용해 재구성시킬 수 있다.

<br>

이때 압축, 즉 재구성은 기존 디스크에 저장됐던 AOF 파일을 사용하는 것이 아니라 레디스 메모리에 있는 데이터를 읽어와서 새로운 파일로 저장하는 형태로 동작한다.  
설정 파일에서 기본 옵션인 `aof-use-rdb-preamble yes`를 `no`로 변경하지 않는다면 이 데이터는 RDB 파일 형태로 저장한다.  
RDB 파일을 저장할 때와 마찬가지로 AOF 파일을 재구성할 때에도 `fork`를 이용해 자식 프로세스를 생성하며, 이 자식 프로세스가 AOF 파일을 재구성해 저장한다.

<br>

#### 📖 레디스 버전 7 이전의 AOF

![aof_ver_7.jpeg](../res/aof_ver_7.jpeg)  

위 그림은 버전 7 이전까지 AOF 파일로, 하나의 파일로 저장됐다.  
- AOF 파일의 앞 부분은 메모리의 데이터를 읽어와 바이너리 형태로 저장한 RDB 파일이 위치한다.
- 이후 레디스의 메모리를 변경한 커맨드 로그들은 RESP 형태로 RDB 파일의 뒤에 쌓이는 형태로 증가한다.

<br>

버전 7 이전까지 AOF 파일이 재구성되는 과정을 하기와 같다.  
![aof_rewrite_ver_7.jpeg](../res/aof_rewrite_ver_7.jpeg)  
1. 레디스는 `fork`를 이용해 자식 프로세스를 생성한다.  
   생성된 자식 프로세스는 레디스 메모리의 데이터를 읽어와 신규로 생성한 임시 파일(RDB)에 저장한다.
2. 백그라운드로 1. 과정이 진행되는 동안 레디스 메모리의 데이터가 변경된 내역은 기존의 AOF 파일과 인메모리 버퍼에 동시에 저장된다.  
   참고로 인메모리 버퍼는 재구성 시에 생긴다.
3. 1.의 AOF 재구성이 끝나면 인메모리 버퍼에 저장된 내용은 1.의 임시 파일 마지막에 추가한다.
4. 생성된 임시 파일로 기존 AOF 파일을 덮어씌운다.

이때 2의 과정에서 RDB 파일이 저장되는 동안 데이터가 변경된 동일한 로그가 AOF 파일과 인메모리 버퍼에 이중으로 저장된다.  
또한 하나의 AOF 파일 내에 바이너리 형태와 RESP 텍스트 형태의 데이터가 함께 저장돼 수동으로 AOF 파일을 처리할 때 관리가 복잡할 수 있다는 단점이 존재한다.

<br>

#### 📖 레디스 버전 7 이후의 AOF

![aof_ver_gt_7.jpeg](../res/aof_ver_gt_7.jpeg)  

버전 7 이후에서 AOF는 기본이 되는 바이너리 형태의 RDB 파일, 증가하는 RESP의 텍스트 형태의 AOF 파일로 나눠서 데이터를 관리한다.  
또한 현재 레디스가 바라보고 있는 파일이 어떤 것인지 나타내는 매니페스트 파일을 추가적으로 도입했으며, 매니페스트 파일은 RDB와 AOF 파일이 어떤 것인지 알려주는 역할을 한다.  
세 파일은 모두 설정 파일에 지정한 `appenddirname` 이름의 폴더 내에 저장된다.

AOF가 재구성될 때마다 AOF를 구성하고 있는 각 RDB와 AOF의 파일명의 번호, 그리고 매니페스트 파일 내부의 `seq` 값도 1씩 증가한다.  

```shell
redis@redisvm:~/redis/appendonlydir$ pwd
/home/redis/redis/appendonlydir
redis@redisvm:~/redis/appendonlydir$ ll
total 20
drwxr-xr-x  2 redis redis 4096 Dec 17 23:34 ./
drwxrwxr-x 11 redis redis 4096 Dec 17 22:48 ../
-rw-rw-r--  1 redis redis 2946 Dec 17 22:46 appendonly.aof.1.base.rdb
-rw-r--r--  1 redis redis  284 Dec 17 23:02 appendonly.aof.1.incr.aof
-rw-r--r--  1 redis redis  114 Dec 17 23:34 appendonly.aof.manifest
redis@redisvm:~/redis/appendonlydir$ cat appendonly.aof.manifest
file appendonly.aof.1.base.rdb seq 1 type b
file appendonly.aof.1.incr.aof seq 1 type i startoffset 0 endoffset 7
```

<br>

![aof_rewrite_gt_7.jpeg](../res/aof_rewrite_gt_7.jpeg)  
1. 레디스 인스턴스는 `fork`를 이용해 자식 프로세스를 생성한다.  
   생성된 자식 프로세스는 레디스 메모리의 데이터를 읽어와 신규로 생성한 임시 파일에 저장한다.
2. 백그라운드로 1.의 과정이 진행되는 동안 레디스 메모리의 데이터가 변경된 내역은 신규 AOF 파일에 저장된다.
3. 1의 AOF 재구성 과정이 끝나면 임시 매니페스트 파일을 생성한 뒤, 변경된 버전으로 매니페스트 파일 내용을 업데이트 한다.
4. 생성된 임시 매니페스트 파일로 기존 매니페스트 파일을 덮어 씌운 뒤, 이전 버전의 AOF, RDB 파일들을 삭제한다.

➡ 기존 버전의 2,3 단계의 비효율을 줄일 수 있어 훨씬 간단한 과정으로 데이터를 저장할 수 있음을 확인할 수 있다.

<br>

앞서 `aof-use-rdb-preamble` 옵션에 의해 압축되는 데이터 파일은 RDB 형태로 저장된다고 했는데, `no`로 옵션값을 설정한다면 베이스 파일은 `*.base.rdb` 형태가 아닌 `*.base.aof` 이름으로 저장되며, 저장되는 형태도 RESP 프로토콜 형태의 텍스트로 변경된다.

<br>

레디스에서 AOF 파일의 재구성 과정은 모두 순차 입출력(sequential I/O)만 사용하기 때문에 디스크에 접근하는 모든 과정이 굉장히 효율적이다.  
레디스의 서버는 복원 시 순차적으로 데이터를 로드하는 용도로만 AOF 파일을 사용한다.  
➡ 파일 내에서 직접 데이터를 검색할 필요가 없기 때문에 랜덤 입출력(random I/O)을 고려할 이유가 전혀 없다.  
RDB 파일을 저장할 때도 마찬가지다.  
파일을 저장할 때 랜덤 입출력을 고려하지 않는다는 점은 모든 데이터 저장소에서 굉장히 드문 기능이다.  

<br>

### ✅ 자동 AOF 재구성
```text
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```
- `auto-aof-rewrite-percentage`: AOF 파일을 다시 쓰기 위한 시점을 정하는 옵션
  - 마지막으로 재구성됐던 AOF 파일의 크기와 비교해, 현재의 AOF 파일이 지정된 퍼센트만큼 커졌을 때 재구성을 시도한다.

```text
127.0.0.1:6379> INFO Persistence
# Persistence
...
# 현재 사이즈
aof_current_size:3230
# base 사이즈
aof_base_size:2946
...
```

만약 `aof_current_size`의 크기가 `2946`의 100%인 `5992`가 된다면 자동으로 재구성을 시도하게 된다.  

데이터가 아무것도 없는 상태로 인스턴스가 처음 부팅됐을 때의 `aof_base_size`는 0이므로, 이럴 때에는 `auto_aof_rewrite_min_size`를 기준으로 데이터를 재구성한다.  
➡ 마지막으로 작성된 AOF 파일 크기를 기준으로 재구성하되, 적어도 AOF 파일이 특정 크기 이상일 때만 재구성을 하도록 지정해 비효율적인 작업을 최소화할 수 있다.

<br>

### ✅ 수동 AOF 재구성
`BGREWRITEAOF` 커맨드를 이용하면 원하는 시점에 직접 AOF 파일을 재구성할 수 있다.  
자동으로 재구성할 때와 동일하게 동작한다.

<br>

### ✅ AOF 타임스탬프
버전 7 이상부터는 AOF를 저장할 때 타임스탬프를 남길 수 있다.  
```text
aof-timestamp-enabled no
```
설정 파일에서 `aof-timestamp-enabled` 옵션을 활성화시키면 다음과 같이 AOF 데이터가 저장될 때 타임스탬프도 함께 저장된다.  

```redis
127.0.0.1:6379> CONFIG GET aof-timestamp-enabled
1) "aof-timestamp-enabled"
2) "no"
127.0.0.1:6379> CONFIG SET aof-timestamp-enabled yes
OK
127.0.0.1:6379> CONFIG REWRITE
OK
127.0.0.1:6379> SET key1 world
OK
```
```text
//  ~/redis/appendonlydir/appendonly.aof.1.incr.aof 내용
#TS:1766065720
*2
$6
SELECT
$1
0
*3
$3
SET
$4
key1
$5
world
```

<br>

#### 🤦🏻‍♂️ AOF 파일을 특정 시점으로 복원하기

다음과 같이 `FLUSHALL` 커맨드를 통해 메모리의 모든 데이터가 날아갔다.
```redis
127.0.0.1:6379> FLUSHALL
OK
```

```text
//  ~/redis/appendonlydir/appendonly.aof.1.incr.aof 내용
#TS:1766065720
*2
$6
SELECT
$1
0
*3
$3
SET
$4
key1
$5
world
#TS:1766095189
*2
$6
SELECT
$1
0
*1
$8
FLUSHALL
```

<br>

리눅스 타임스탬프를 `1766095188`(1766095189-1)로 돌려보자.  
```shell
redis@redisvm:~/redis$ src/redis-check-aof --truncate-to-timestamp 1766095188 appendonlydir/appendonly.aof.manifest
Start checking Multi Part AOF
Start to check BASE AOF (RDB format).
[offset 0] Checking RDB file appendonlydir/appendonly.aof.1.base.rdb
[offset 26] AUX FIELD redis-ver = '8.2.3'
[offset 40] AUX FIELD redis-bits = '64'
[offset 52] AUX FIELD ctime = '1766011565'
[offset 67] AUX FIELD used-mem = '867552'
[offset 79] AUX FIELD aof-base = '1'
[offset 81] Selecting DB ID 0
[offset 2946] Checksum OK
[offset 2946] \o/ RDB looks OK! \o/
[info] 55 keys read
[info] 0 expires
[info] 0 already expired
[info] 0 subexpires
RDB preamble is OK, proceeding with AOF tail...
Truncate nothing in AOF appendonly.aof.1.base.rdb to timestamp 1766095188
BASE AOF appendonly.aof.1.base.rdb is valid
Start to check INCR files.
Successfully truncated AOF appendonly.aof.1.incr.aof to timestamp 1766095188
All AOF files and manifest are valid
```

<br>

AOF 파일을 다시 확인해보면 `FLUSHALL`을 수행하기 전의 내용만 남아있음을 확인할 수 있다.
```text
//  ~/redis/appendonlydir/appendonly.aof.1.incr.aof 내용
#TS:1766065720
*2
$6
SELECT
$1
0
*3
$3
SET
$4
key1
$5
world
         
```

<br>

`--truncate-to-timestamp` 옵션을 사용해 AOF 파일을 복구하면 원본 파일이 변경된다.  
➡ 작업을 수행하기 이전의 AOF 파일을 보호하고 싶다면 원본 파일을 미리 다른 곳에 복사해두는 것이 좋다.

<br>

### ✅ AOF 파일 복원
시점 복원(point-in-time recovery)에서 사용한 `redis-check-aof` 프로그램은 AOF 파일이 손상됐을 때에도 사용할 수 있다.  
의도치 않은 서버의 장애 발생 시 AOF 파일 작성 도중 레디스가 중지됐을 가능성이 존재한다.  
레디스가 의도치 않은 장애로 중단됐을 때 `redis-check-aof` 프로그램은 AOF 파일의 상태가 정상적인지 확인할 수 있다.
```shell
redis@redisvm:~/redis$ src/redis-check-aof appendonlydir/appendonly.aof.manifest
Start checking Multi Part AOF
Start to check BASE AOF (RDB format).
[offset 0] Checking RDB file appendonlydir/appendonly.aof.1.base.rdb
[offset 26] AUX FIELD redis-ver = '8.2.3'
[offset 40] AUX FIELD redis-bits = '64'
[offset 52] AUX FIELD ctime = '1766011565'
[offset 67] AUX FIELD used-mem = '867552'
[offset 79] AUX FIELD aof-base = '1'
[offset 81] Selecting DB ID 0
[offset 2946] Checksum OK
[offset 2946] \o/ RDB looks OK! \o/
[info] 55 keys read
[info] 0 expires
[info] 0 already expired
[info] 0 subexpires
RDB preamble is OK, proceeding with AOF tail...
AOF analyzed: filename=appendonly.aof.1.base.rdb, size=2946, ok_up_to=2946, ok_up_to_line=1, diff=0
BASE AOF appendonly.aof.1.base.rdb is valid
Start to check INCR files.
AOF analyzed: filename=appendonly.aof.1.incr.aof, size=357, ok_up_to=357, ok_up_to_line=73, diff=0
INCR AOF appendonly.aof.1.incr.aof is valid
All AOF files and manifest are valid
```
RDB와 AOF 파일이 모두 정상이다.  
만약 AOF 파일이 비정상이면 `AOF appendonly.aof.1.incr.aof is not valid. Use the --fix option to try fixing it.`이라는 문구가 제일 마지막 줄에 나올 것이다.

<br>

```shell
redis@redisvm:~/redis$ src/redis-check-aof --fix appendonlydir/appendonly.aof.manifest
Start checking Multi Part AOF
Start to check BASE AOF (RDB format).
[offset 0] Checking RDB file appendonlydir/appendonly.aof.1.base.rdb
[offset 26] AUX FIELD redis-ver = '8.2.3'
[offset 40] AUX FIELD redis-bits = '64'
[offset 52] AUX FIELD ctime = '1766011565'
[offset 67] AUX FIELD used-mem = '867552'
[offset 79] AUX FIELD aof-base = '1'
[offset 81] Selecting DB ID 0
[offset 2946] Checksum OK
[offset 2946] \o/ RDB looks OK! \o/
[info] 55 keys read
[info] 0 expires
[info] 0 already expired
[info] 0 subexpires
RDB preamble is OK, proceeding with AOF tail...
AOF analyzed: filename=appendonly.aof.1.base.rdb, size=2946, ok_up_to=2946, ok_up_to_line=1, diff=0
BASE AOF appendonly.aof.1.base.rdb is valid
Start to check INCR files.
AOF analyzed: filename=appendonly.aof.1.incr.aof, size=357, ok_up_to=357, ok_up_to_line=73, diff=0
INCR AOF appendonly.aof.1.incr.aof is valid
All AOF files and manifest are valid
```
`fix` 옵션을 사용한 복구 또한 원본 파일을 변경하기 때문에 이전의 데이터를 보호하고 싶은 경우에는 원본 데이터를 다른 곳에 복사해두는 것이 안전하다.

<br>

### ✅ AOF 파일의 안전성
시점 단위로 데이터를 저장하는 RDB 방식보다 명령어 단위로 로그를 저장하는 AOF 방식이 더 안전하다.  
운영체제에서 시스템 콜을 이용해 데이터를 파일에 저장하는 방법을 먼저 간단히 보자.  

![redis_data_to_disk.jpeg](../res/redis_data_to_disk.jpeg)  
운영체제에서 애플리케이션이 파일에 데이터를 저장하고자 할 때, 곧바로 디스크에 데이터가 저장되진 않는다.  
➡ `WRITE`라는 시스템 콜을 이용해 애플리케이션에서 파일에 데이터를 저장하겠다 하면 데이터는 커널 영역의 OS 버퍼에 임시로 저장한다.  
➡ 운영체제가 판단하기에 커널이 여유 있거나, 최대 지연 시간인 30초에 도달하면 커널 버퍼의 데이터를 실제로 디스크에 내려 쓴다.

<br>

`FSYNC`는 커널의 OS 버퍼에 저장된 내용을 실제로 디스크에 내리도록 강제하는 시스템 콜이다.  
OS에 부하가 있더라도 `FSYNC`가 호출되면 데이터는 무조건 디스크에 `flush` 된다.

<br>

#### 💡 `APPENDFSYNC` 옵션
레디스에서 AOF 파일을 저장할 때 `APPENDFSYNC` 옵션을 이용하면 `FSYNC` 호출을 제어할 수 있다.  
즉, 파일 저장의 내구성을 제어할 수 있다.

- `APPENDFSYNC no`
  - AOF 데이터를 저장할 때 WRITE 시스템 콜을 호출한다.
  - 데이터는 커널 영역에 데이터가 잘 저장되는지만 확인하기 때문에 쓰기 성능이 가장 빠르다.
  - 서버에 장애가 발생하면 최대 30초 동안 레디스에 입력했던 데이터를 잃을 수 있다.
- `APPENDFSYNC always`
  - AOF 데이터를 저장할 때 항상 WRITE와 FSYNC 시스템 콜을 함께 호출한다.
  - 즉, 매번 쓰고자 하는 데이터가 파일에 정확하게 저장되는 것을 기다리기 때문에 쓰기 성능은 가장 느리다.
- `APPENDFSYNC everysec`
  - 데이터를 저장할 때 WRITE 시스템 콜을 호출하며, 1초에 한 번씩 FSYNC 시스템 콜을 호출한다.
  - 성능은 `no` 옵션을 사용했을 때와 거의 비슷하다.
  - 디폴트 옵션 값으로, 속도와 안전성의 균형을 맞출 수 있는 값이기 때문에 권장한다.

<br>

## ❗️ 백업을 사용할 때 주의할 점
RDB와 AOF 파일을 사용하는 경우 인스턴스의 `maxmemory` 값은 실제 서버 메모리보다 여유를 갖고 설정하는 것이 좋다.

`BGSAVE` 커맨드로 RDB 파일을 저장하거나 AOF 재구성을 진행할 때 레디스는 `fork()`를 이용해 자식 프로세스를 생성한다.  
➡ 생성된 자식 프로세스는 레디스의 메모리를 그대로 파일에 저장해야 하며, 기존의 부모 프로세스는 다른 메모리의 데이터를 이용해 다른 클라이언트의 연결을 처리해야 한다.  
➡ 이때 레디스는 `Copy-On-Write` 방식을 이용해 메모리상의 데이터를 하나 더 복사하는 방법을 이용해 백업을 진행하면서도 클라이언트의 요청 사항을 받아 메모리의 데이터를 읽고 수정하는 작업을 진행할 수 있다.

<br>

![copy_on_write.jpeg](../res/copy_on_write.jpeg)  

위 그림과 같이 물리적 메모리에 있는 실제 메모리 페이지(`page C`)가 그대로 복제되기 때문에 최악의 경우 레디스는 기존 메모리 용량의 2배를 사용하게 될 수도 있다.  
➡ 레디스의 `maxmemory` 값을 너무 크게 설정한 경우, 레디스의 `copy-on-write` 동작으로 인해 OS 메모리가 가득 차는 상황이 발생할 수 있으며, 이로 인해 `OOM(Out Of Memory)` 문제로 서버가 다운될 수 있다.

<br>

따라서 레디스의 `maxmemory` 옵션은 실제 메모리보다 여유를 갖고 설정하는 것이 안정적이다.  
다음 표와 같이 서버의 메모리 유형에 따라 적절한 `maxmemory` 값을 지정하는 것이 좋다.

| RAM  | Maxmemory | 비율  |
|:----:|:---------:|:---:|
| 2GB  |   638MB   | 33% |
| 4GB  |  2048MB   | 50% |
| 8GB  |  4779MB   | 58% |
| 16GB |  10240MB  | 63% |
| 32GB |  21163MB  | 65% |
| 64GB |  43008MB  | 66% |

<br>

RDB 스냅숏을 저장하는 도중에는 AOF의 재구성 기능을 사용할 수 없고, AOF 재구성이 진행될 때에는 `BGSAVE`를 실행할 수 없다.




<br>

**참고 자료**  
[개발자를 위한 레디스](https://product.kyobobook.co.kr/detail/S000210785682)