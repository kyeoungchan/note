# 💻 레디스 기본 개념
데이터는 레디스에서 제공하는 다양한 형태의 자료 구조로 저장될 수 있다.

## ✅ 레디스의 자료 구조
### ❗️ String
- `string`은 레디스에서 데이터를 저장할 수 있는 가장 간단한 자료구조다.  
- `string`에는 최대 512MB의 문자열 데이터를 저장할 수 있으며, 이진 데이터를 포함한 모든 종류의 문자열이 binary-safe하게 처리되기 때문에 JPEG 이미지와 같은 바이트 값, HTTP 응답값 등의 다양한 데이터를 저장하는 것도 가능하다.
- `string`은 키와 실제 저장되는 아이템이 일대일로 연결되는 유일한 자료 구조이며, `string`이 아닌 다른 자료 구조에서는 하나의 키에 여러 개의 아이템이 저장된다.

```shell
# hello 키에 world라는 string 데이터 연결 
127.0.0.1:6379> SET hello world
OK
127.0.0.1:6379> GET hello
"world"
```

<br>

```shell
# NX 옵션은 지정한 키가 없을 때에만 새로운 키를 저장한다.
127.0.0.1:6379> SET hello newval NX
(nil)
```

<br>

```shell
# XX 옵션은 반대로 키가 이미 있을 때에만 새로운 값으로 덮어 쓴다.
127.0.0.1:6379> SET hello newval XX
OK
127.0.0.1:6379> GET hello
"newval"
```

<br>

```shell
# string 자료 구조에 저장된 숫자를 원자적(atomic)으로 조작할 수 있다.
127.0.0.1:6379> SET counter 100
OK
# 1씩 데이터 증가
127.0.0.1:6379> INCR counter
(integer) 101
127.0.0.1:6379> INCR counter
(integer) 102
# 입력한 값만큼 데이터 증가
127.0.0.1:6379> INCRBY counter 50
(integer) 152
# 1씩 데이터 감소
127.0.0.1:6379> DECR counter
(integer) 151
# 입력한 값만큼 데이터 감소
127.0.0.1:6379> DECRBY counter 20
(integer) 131
```
> 커맨드가 원자적이라는 것은 같은 키에 접근하는 여러 클라이언트가 경쟁 상태(race condition)를 발생시킬 일이 없음을 의미한다.  
> 커맨드를 수행하는 타이밍이나 순서에 따라 이미 실행한 커맨드가 무시되거나 같은 커맨드가 중복 처리돼 수행 결과가 달라지는 일이 발생하지 않음을 뜻한다.

<br>

```shell
# 한 번에 여러 키 조작
127.0.0.1:6379> MSET a 10 b 20 c 30
OK
127.0.0.1:6379> MGET a b c
1) "10"
2) "20"
3) "30"
```
`MGET`과 `MSET`과 같은 커맨드를 적절하게 사용해 네트워크 통신 시간을 줄인다면 전반적으로 서비스의 응답 속도를 확실하게 향상시킬 수 있다.

<br>

### ❗️ list
- `list`는 순서를 가지는 문자열의 목록이다.
- 하나의 `list`에는 최대 42억여 개의 아이템을 저장할 수 있다.
- 다른 배열처럼 인덱스를 이용해 데이터에 직접 접근할 수 있고, 일반적으로는 스택과 큐로서 사용된다.

```shell
# list의 왼쪽에 데이터를 추가한다.
127.0.0.1:6379> LPUSH mylist E
(integer) 1
# list의 오른쪽에 데이터를 추가한다.
127.0.0.1:6379> RPUSH mylist B
(integer) 2
# 여러 아이템을 저장하는 것도 가능하며, 아이템은 나열된 순서대로 하나씩 list에 저장된다.
127.0.0.1:6379> LPUSH mylist D A C B A
(integer) 7
# 시작과 끝 아이템의 인덱스를 각각 인수로 받아 출력한다.
# 가장 오른쪽(tail)에 있는 아이템의 인덱스는 -1이다.
127.0.0.1:6379> LRANGE mylist 0 -1
1) "A"
2) "B"
3) "C"
4) "A"
5) "D"
6) "E"
7) "B"
127.0.0.1:6379> LRANGE mylist 0 3
1) "A"
2) "B"
3) "C"
4) "A"
```

<br>

```shell
# list에 저장된 첫 번째 아이템을 반환하는 동시에 list에서 삭제한다.
127.0.0.1:6379> LPOP mylist
"A"
# 지정한 숫자만큼의 아이템을 반복해서 반환한다.
127.0.0.1:6379> LPOP mylist 2
1) "B"
2) "C"
```

<br>

```shell
127.0.0.1:6379> LRANGE mylist 0 -1
1) "A"
2) "D"
3) "E"
4) "B"
# 인자로 전달받은 인덱스 범위에 속하지 않은 아이템을 모두 삭제한다.
# 삭제되는 아이템을 반환하지는 않는다.
127.0.0.1:6379> LTRIM mylist 0 1
OK
127.0.0.1:6379> LRANGE mylist 0 -1
1) "A"
2) "D"
```

> **`LPUSH`와 `LTRIM` 커맨드를 함께 사용하여 고정된 길이의 큐 유지 ➡ 로그 관리**  
> 로그는 계속해서 쌓이는 데이터이기 때문에 주기적으로 로그 데이터를 삭제해 저장 공간을 확보하는 것이 일반적이다. 다음과 같이 커맨드를 쓸 수 있다.  
> `LPUSH logdata <data>`  
> `LTRIM logdata 0 999`  
> list에 데이터를 저장하면서 매번 1,000번째 이상의 인덱스를 삭제한다.  
> 로그 데이터를 일단 쌓은 뒤 주기적으로 배치 처리를 이용해 삭제하는 것보다 훨씬 효율적이다.  
> 매번 큐의 마지막 데이터만 삭제되기 때문인데, list에서 tail의 데이터를 삭제하는 작업은 O(1)로 동작한다.  
> 참고로 인덱스나 데이터를 이용해 list의 중간 데이터에 접근할 때에는 O(n)으로 처리되며, list에 저장된 데이터가 늘어남에 따라 성능은 저하된다.

<br>

```shell
# 데이터의 앞에 추가하려면 BEFORE, 뒤에 추가하려면 AFTER 옵션을 추가한다.
# 만약 지정한 데이터가 없으면 오류를 반환한다.
127.0.0.1:6379> LINSERT mylist BEFORE D E
(integer) 3
127.0.0.1:6379> LRANGE mylist 0 -1
1) "A"
2) "E"
3) "D"

# 지정한 인덱스의 데이터를 신규 입력하는 데이터로 덮어 쓴다.
127.0.0.1:6379> LSET mylist 2 C
OK
127.0.0.1:6379> LRANGE mylist 0 -1
1) "A"
2) "E"
3) "C"

# 원하는 인덱스의 데이터를 확인할 수 있다.
127.0.0.1:6379> LINDEX mylist 2
"C"
```

<br>

### ❗️ hash
`hash`는 필드-값 쌍을 가진 아이템의 집합이다.  
`hash`는 객체를 표헌하기 적절한 자료구조이기 때문에 관계형 데이터베이스의 테이블 데이터로 변환하는 것도 간편하다.

<br>

```shell
# 아이템 저장
127.0.0.1:6379> HSET Product:123 Name "Happy Hacking"
(integer) 1
127.0.0.1:6379> HSET Product:123 TypeID 35
(integer) 1
127.0.0.1:6379> HSET Product:123 Version 2002
(integer) 1
# 한 번에 여러 필드-값 쌍 저장
127.0.0.1:6379> HSET Product:234 Name "TrackBall" TypeID 32
(integer) 2
```

<br>

`HGET` 커맨드로 가져올 때 `hash` 자료 구조의 키와 아이템의 필드를 함께 입력해야한다.
```shell
127.0.0.1:6379> HGET Product:123 TypeID
"35"
# 하나의 hash 내에서 다양한 필드의 값 가져온다.
127.0.0.1:6379> HMGET Product:234 Name TypeID
1) "TrackBall"
2) "32"
# hash 내의 모든 필드-값 쌍을 차례로 반환한다.
127.0.0.1:6379> HGETALL Product:234
1) "Name"
2) "TrackBall"
3) "TypeID"
4) "32"
```

<br>

### ❗️set
레디스에서 `set`은 정렬되지 않은 문자열의 모음이다.  
객체 간의 관계를 계산하거나 유일한 원소를 구해야할 경우에 사용될 수 있다.  

```shell
# set에 아이템 저장
127.0.0.1:6379> SADD myset A
(integer) 1
# 한 번에 여러 개의 아이템 저장
127.0.0.1:6379> SADD myset A A A C B D D E F F F G
(integer) 6
# set에 저장된 전체 아이템 출력
127.0.0.1:6379> SMEMBERS myset
1) "A"
2) "C"
3) "B"
4) "D"
5) "E"
6) "F"
7) "G"
```

<br>

```shell
# 원하는 데이터 삭제
127.0.0.1:6379> SREM myset B
(integer) 1
# 랜덤으로 하나의 아이템을 반환함과 동시에 삭제
127.0.0.1:6379> SPOP myset
"C"
```

> 상황  
> set:111 : {A, B, C, D, E}  
> set:222 : {D, E, F, G, H}

```shell
# set:111과 set:222 셋팅
127.0.0.1:6379> SADD set:111 A B C D E
(integer) 5
127.0.0.1:6379> SADD set:222 D E F G H
(integer) 5

# 교집합 
127.0.0.1:6379> SINTER set:111 set:222
1) "D"
2) "E"

# 합집합
127.0.0.1:6379> SUNION set:111 set:222
1) "G"
2) "H"
3) "F"
4) "C"
5) "E"
6) "D"
7) "A"
8) "B"

# 차집합(set:111 - set:222)
127.0.0.1:6379> SDIFF set:111 set:222
1) "C"
2) "B"
3) "A"
```

<br>

### ❗️ sorted set
- `sorted set`은 스코어 값에 따라 정렬되는 고유한 문자열의 집합이다.  
- 모든 아이템은 스코어-값 쌍을 가지며, 저장될 때부터 스코어 앖으로 정렬돼 저장된다.
- 같은 스코어를 가진 아이템은 데이터의 사전 순으로 정렬돼 저장된다.
- 모든 아이템은 스코어 순으로 정렬돼 있으므로 `list`처럼 인덱스를 이용해 각 아이템에 접근할 수 있다.
> `list`와 `sorted set` 모두 순서를 갖는 자료구조이므로 인덱스를 이용해 아이템에 접근할 수 있다.  
> 인덱스를 이용해 아이템에 접근할 일이 많다면 `list`가 아닌 `sorted set`을 사용하는 것이 더 효율적이다.  
> `list`에서 인덱스를 이용해 데이터를 접근하는 것은 O(n)으로 처리되지만, `sorted set`에서는 O(long(n))으로 처리되기 때문이다.

```shell
127.0.0.1:6379> ZADD score:251113 100 user:B
(integer) 1
127.0.0.1:6379> ZADD score:251113 150 user:A 150 user:C 200 user:F 300 user:E
(integer) 4
```

`ZADD` 커맨드 옵션
- XX: 아이템이 이미 존재할 때에만 스코어를 업데이트한다.
- NX: 아이템이 존재하지 않을 때에만 신규 삽입하며, 기존 아이템의 스코어를 업데이트하지 않는다.
- LT: 업데이트하고자 하는 스코어가 기존 아이템의 스코어보다 작을 대에만 업데이트 한다. 기존에 아이템이 존재하지 않을 때에는 새로운 데이터를 삽입한다.
- GT: 업데이트하고자 하는 스코어가 기존 아이템의 스코어보다 클 때에만 업데이트 한다. 기존에 아이템이 존재하지 않을 때에는 새로운 데이터를 삽입한다. 

<br>

`ZRANGE` 커맨드를 사용하려면 `start`와 `stop`이라는 범위를 항상 입력해야 한다.  
```shell
ZRANGE key start stop [BYSCORE | BYINDEX] [REV] [LIMIT offset count] [WITHSCORES]
```

<br>


**인덱스로 데이터 조회**  
```shell
# 1~3번 인덱스 조회
127.0.0.1:6379> ZRANGE score:251113 1 3 WITHSCORES
1) "user:A"
2) "150"
3) "user:C"
4) "150"
5) "user:F"
6) "200"

# 1~3번 인덱스 역순 조회
127.0.0.1:6379> ZRANGE score:251113 1 3 WITHSCORES REV
1) "user:F"
2) "200"
3) "user:C"
4) "150"
5) "user:A"

# 전체 조회
127.0.0.1:6379> ZRANGE score:251113 0 -1 WITHSCORES
 1) "user:B"
 2) "100"
 3) "user:A"
 4) "150"
 5) "user:C"
 6) "150"
 7) "user:F"
 8) "200"
 9) "user:E"
10) "300"
```

<br>

**스코어로 데이터 조회**  
```shell
# 100 이상 150 이하 스코어로 조회
127.0.0.1:6379> ZRANGE score:251113 100 150 BYSCORE WITHSCORES
1) "user:B"
2) "100"
3) "user:A"
4) "150"
5) "user:C"
6) "150"

# 10 이상 150 이하 스코어로 조회
127.0.0.1:6379> ZRANGE score:251113 10 150 BYSCORE WITHSCORES
1) "user:B"
2) "100"
3) "user:A"
4) "150"
5) "user:C"
6) "150"

# 최소값이 최대값보다 크게 조회를 시켜도 에러는 나지 않는다. 빈 배열 출력
127.0.0.1:6379> ZRANGE score:251113 310 150 BYSCORE WITHSCORES
(empty array)

# 당연히 존재하지 않는 범위로 조회하면 빈 배열이 출력된다.
127.0.0.1:6379> ZRANGE score:251113 310 450 BYSCORE WITHSCORES
(empty array)

# 100 초과 150 이하 스코어로 조회
127.0.0.1:6379> ZRANGE score:251113 (100 150 BYSCORE WITHSCORES
1) "user:A"
2) "150"
3) "user:C"
4) "150"

# 100 이상 150 미만 스코어로 조회
127.0.0.1:6379> ZRANGE score:251113 100 (150 BYSCORE WITHSCORES
1) "user:B"
2) "100"

# 200 이상 스코어로 조회
# +inf는 최댓값
127.0.0.1:6379> ZRANGE score:251113 200 +inf BYSCORE WITHSCORES
1) "user:F"
2) "200"
3) "user:E"
4) "300"

# 모든 스코어로 조회
# -inf는 최솟값
127.0.0.1:6379> ZRANGE score:251113 -inf +inf BYSCORE WITHSCORES
 1) "user:B"
 2) "100"
 3) "user:A"
 4) "150"
 5) "user:C"
 6) "150"
 7) "user:F"
 8) "200"
 9) "user:E"
10) "300"

# 200 이상 스코어 역순으로 조회
# 역순 조회시 최솟값과 최댓값 스코어의 전달 순서를 바꿔야한다.
127.0.0.1:6379> ZRANGE score:251113 +inf 200 BYSCORE WITHSCORES REV
1) "user:E"
2) "300"
3) "user:F"
4) "200"
```

<br>

**사전 순으로 데이터 조회**  
- 데이터를 저장할 때 스코어가 같으면 데이터는 사전 순으로 정렬된다.
- 스코어가 같을 때 `BYLEX` 옵션을 사용하면 사전식 순서를 이용해 아이템을 조회할 수 있다.
- 문자열은 ASCII 바이트 값에 따라 사전식으로 정렬되기 때문에, 한글 문자열도 이 기준에 따라 정렬하거나 사전식으로 검색할 수 있다.
```shell
# mySortedSet 셋팅
127.0.0.1:6379> ZADD mySortedSet 0 apple 0 banana 0 candy 0 dream 0 egg 0 frog
(integer) 6

# b 이상, f 이하 아이템 조회
127.0.0.1:6379> ZRANGE mySortedSet (b (f BYLEX
1) "banana"
2) "candy"
3) "dream"
4) "egg"
```
- 입력한 문자열을 포함하려면 `(`을, 포함하지 않을 때에는 `[` 문자를 사용한다.
- 사전식 문자열의 가장 처음은 `-` 문자로, 가장 마지막은 `+` 문자로 대체할 수 있다.
```shell
# 점수가 같지 않은 아이템이 온다면 어떨까?
127.0.0.1:6379> ZADD mySortedSet 10 cream
(integer) 1

# b 이상, f 이하로 아이템을 조회해도 cream이 조회가 되지 않는다.
127.0.0.1:6379> ZRANGE mySortedSet (b (f BYLEX
1) "banana"
2) "candy"
3) "dream"
4) "egg"

# 전체를 조회해보면 cream이 제일 뒤에 가있다.
# 기본으로 스코어 순으로 정렬 후, 문자열 사전식 정렬 순으로 조회가 된다.
127.0.0.1:6379> ZRANGE mySortedSet - + BYLEX
1) "apple"
2) "banana"
3) "candy"
4) "dream"
5) "egg"
6) "frog"
7) "cream"

# cream 멤버 삭제
127.0.0.1:6379> ZREM mySortedSet cream
(integer) 1
```

<br>

### ❗️ 비트맵
비트맵은 string 자료 구조에 bit 연산을 수행할 수 있도록 확장한 형태다.
- string 자료구조가 binary safe하고 최대 512MB의 값을 저장할 수 있는 구조이기 때문에 2^32의 비트를 가지고 있는 비트맵 형태다.
- 비트맵을 사용할 때의 가장 큰 장점은 저장 공간을 획기적으로 줄일 수 있다는 것이다.
  - 예를 들어 각각의 유저가 정수 형태의 ID로 구분되고, 전체 유저가 40억이 넘는다고 해도 각 유저에 대한 y/n 데이터는 512MB 안에 충분히 저장할 수 있다.

```shell
# SETBIT 커맨드로 비트 저장
127.0.0.1:6379> SETBIT mybitmap 2 1
(integer) 0

# GETBIT 커맨드로 저장된 비트 조회
127.0.0.1:6379> GETBIT mybitmap 2
(integer) 1
127.0.0.1:6379> GETBIT mybitmap 1
(integer) 0

# BITFIELD 커맨드로 여러 비트를 SET
127.0.0.1:6379> BITFIELD mybitmap SET u1 6 1 SET u1 10 1 SET u1 14 1
1) (integer) 0
2) (integer) 0
3) (integer) 0

# BITCOUNT 커맨드로 1로 설정된 비트의 개수 카운팅
127.0.0.1:6379> BITCOUNT mybitmap
(integer) 4
```

<br>

### ❗️ Hyperloglog

`hyperloglog`는 집합의 원소 개수인 카디널리티를 추정할 수 있는 자료 구조다.  
- 대량 데이터에서 중복되지 않는 고유한 값을 집계할 때 유용하게 사용할 수 잇는 데이터 구조다.
- 최대 12KBdml zmrlfmf rkwlsek.
- 추정의 오차는 0.81%로, 비교적 정확하게 데이터를 추정할 수 있다.
- 하나의 `hyperloglog`에는 최대 2^64개의 아이템을 저장할 수 있다.

> 일반적으로 `set`과 같은 데이터 구조에서는 중복을 피하기 위해 저장된 데이터를 모두 기억하고 있어, 저장되는 데이터가 많아질수록 그만큼 많은 메모리를 사용한다.  
> `hyperloglog`는 입력되는 데이터 그 자체를 저장하지 않고 자체적인 방법으로 데이터를 변경해 처리한다.  
> 따라서 저장되는 데이터 개수에 구애받지 않고 계속 일정한 메모리를 유지할 수 있으며, 중복되지 않는 유일한 원소의 개수를 계산할 수 있다. 

```shell
# PFADD 커맨드로 hyperloglog에 아이템 저장
127.0.0.1:6379> PFADD members 123
(integer) 1
127.0.0.1:6379> PFADD members 500
(integer) 1
127.0.0.1:6379> PFADD members 12
(integer) 1

# PFCOUNT 커맨드로 저장된 아이템의 개수, 즉 카디널리티 추정
127.0.0.1:6379> PFCOUNT members
(integer) 3

# 중복된 데이터는 저장되지 않는다.
127.0.0.1:6379> PFADD members 12
(integer) 0
127.0.0.1:6379> PFCOUNT members
(integer) 3
```

<br>

### ❗️ Geospatial
`Geospatial` 자료 구조는 경도, 위도 데이터 쌍의 집합으로 간편하게 지리 데이터를 저장할 수 있는 방법이다.  
내부적으로 데이터는 `sorted set`으로 저장되며, 하나의 자료 구조 안에 키는 중복돼 저장되지 않는다.

```shell
# GEOADD 커맨드를 통해 아이템 추가
# sorted set과 마찬가지로 
#   XX 옵션을 사용하면 이미 아이템이 있는 경우에만
#   NX 옵션을 사용하면 아이템이 없는 경우에만 데이터를 저장한다.
127.0.0.1:6379> GEOADD travel 14.399698 50.099242763 prague
(integer) 1
127.0.0.1:6379> GEOADD travel 127.0016985 37.5642135 seoul -122.434547622 37.78530362582 SanFrancisco
(integer) 2

# GEOPOS 커맨드를 통해 위치 데이터 조회
127.0.0.1:6379> GEOPOS travel seoul
1) 1) "127.00169831514359"
   2) "37.56421398780153"

# GEODIST 커맨드를 통해 두 아이템 사이의 거리 반환
127.0.0.1:6379> GEODIST travel seoul prague KM
"8252.9957"
```

그 외에 커맨드
- `GEOSEARCH`: 특정 위치를 기준으로 원하는 거리 내에 있는 아이템 검색
  - `BYRADIUS` 옵션: 반경 거리 기준
  - `BYBOX` 옵션: 직사각형 거리 기준

<br>

### ❗️ stream
`stream`은 레디스를 메시지 브로커로서 사용할 수 있게 하는 자료 구조다.
- 전체적인 구조는 카프카에서 영향을 받아 만들어졌다.
- 카프카에서처럼 소비자 그룹 개념을 도입해 데이터를 분산 처리할 수 있는 시스템이다.
- 실시간 이벤트 혹은 로그성 데이터의 저장을 위해 사용할 수 있다.

<br>

## ✅ 레디스에서 키를 관리하는 법
### ❗️ 키의 자동 생성과 삭제
`stream`이나 `set`, `sorted set`, `hash`와 같이 하나의 키가 여러 개의 아이템을 가지고 있는 자료 구조에서는 명시적으로 키를 생성하거나 삭제하지 않아도 키는 알아서 생성되고 삭제된다.

1. 키가 존재하지 않을 때 아이템을 넣으면 아이템을 삽입하기 전에 빈 자료 구조를 생성한다.  
    ```shell
    127.0.0.1:6379> DEL mylist
    (integer) 1
    127.0.0.1:6379> LPUSH mylist 1 2 3
    (integer) 3
    ```
   
    저장하고자 하는 키에 다른 자료 구조가 이미 생성돼 있을 때 아이템을 추가하는 작업은 에러를 반환한다.
    ```shell
    127.0.0.1:6379> SET hello wordl
    OK
    127.0.0.1:6379> LPUSH hello 1 2 3
    (error) WRONGTYPE Operation against a key holding the wrong kind of value
    127.0.0.1:6379> TYPE hello
    string
    ```
2. 모든 아이템을 삭제하면 키도 자동으로 삭제된다(`stream`은 예외).
    ```shell
    127.0.0.1:6379> LPUSH mylist 1 2 3
    (integer) 3
    # mylist 존재
    127.0.0.1:6379> EXISTS mylist
    (integer) 1
    127.0.0.1:6379> LPOP mylist
    "3"
    127.0.0.1:6379> LPOP mylist
    "2"
    127.0.0.1:6379> LPOP mylist
    "1"
    # mylist 미존재
    127.0.0.1:6379> EXISTS mylist
    (integer) 0
    ```
3. 키가 없는 상태에서 키 삭제, 아이템 삭제, 자료 구조 크기의 조회 같은 읽기 전용 커맨드를 수행하면 에러를 반환하는 대신 키가 있으나 아이템이 없는 것처럼 동작한다.
    ```shell
    127.0.0.1:6379> DEL mylist
    (integer) 0
    127.0.0.1:6379> LLEN mylist
    (integer) 0
    127.0.0.1:6379> LPOP mylist
    (nil)
    ```

<br>

### ❗️ 키와 관련된 커맨드
**키의 조회**  
`EXISTS`
```shell
127.0.0.1:6379> SET hello world
OK
# 오타도 그냥 적어봤다.
127.0.0.1:6379> EISTS hello
(error) ERR unknown command 'EISTS', with args beginning with: 'hello' 
127.0.0.1:6379> EXISTS hello
(integer) 1
127.0.0.1:6379> EXISTS world
(integer) 0
```

<br>

`KEYS`      
```shell
KEYS pattern
```
매칭되는 패턴에 해당하는 모든 키의 list를 반환한다.  
패턴은 글롭 패턴(Glob Pattern) 스타일로 동작한다.
- h?llo에는 hello, hallo가 매칭될 수 있다.
- h*llo에는 hllo, heeeeello가 매칭될 수 있다.
- h[ae]llo에는 hello, hallo가 매칭될 수 있지만, haello, hillo는 매칭되지 않는다.
  ```shell
  127.0.0.1:6379> SET haello hi
  OK
  127.0.0.1:6379> KEYS h[ae]llo
  1) "hello"
  ```
- h[^e]llo에는 hallo, hbllo가 매칭될 수 있지만, hello는 매칭되지 않는다.
- h[a-c]llo에는 hallo, hbllo, hcllo까지 매칭될 수 있다.
  ```shell
  127.0.0.1:6379> SET hbllo world
  OK
  127.0.0.1:6379> SET hcllo world
  OK
  127.0.0.1:6379> KEYS h[a-c]llo
  1) "hcllo"
  2) "hbllo"
  ```

<br>

`SCAN`  
```redis
SCAN cursor [MATCH pattern] [COUNT count] [TYPE type]
```
`SCAN`은 `KEYS`를 대체해 키를 조회할 때 사용할 수 있는 커맨드다.
- `KEYS`: 한 번에 모든 키를 반환하는 커맨드로, 잘못 사용하면 문제가 발생할 수 있다.
- `SCAN`: 커서를 기반으로 특정 범위의 키만 조회할 수 있기 때문에 비교적 안전하게 사용할 수 있다.

```redis
// 처음으로 SCAN 커맨드 사용시 커서에 0을 입력했다.
> SCAN 0
1) "19"
2)  1) "Product:123"
    2) "set:222"
    3) "b"
    4) "c"
    5) "Product:234"
    6) "counter"
    7) "set:111"
    8) "travel"
    9) "members"
   10) "mybitmap"
   11) "score:251113"

> SCAN 19
1) "0"
2) 1) "a"
   2) "hello"
   3) "myset"
   4) "mySortedSet"
```
- 다음 `SCAN` 커맨드를 사욯라 때 인수로 사용해야하는 커서의 위치가 첫 번째로 반환된다.
  - 0 ➡ 19
- 두 번째 `SCAN` 커맨드는 다음 커서 값으로 0을 반환했으며, 이는 레디스에 저장된 모든 키를 반환해서 더 이상 검색할 키 값이 없다는 것을 의미한다.
  - 클라이언트는 반환되는 첫 번째 인자가 0이 될 때까지 `SCAN`을 반복적으로 사용해 레디스에 저장된 모든 키를 확인할 수 있다.
- 기본적으로 한 번에 반환되는 키의 개수는 10개 정도지만, `COUNT` 옵션을 사용하면 개수를 조정할 수 있다.
  - 하지만 레디스는 메모리를 스캔하며 데이터가 저장된 형상에 따라 몇 개의 키를 더 읽는 것이 효율적이라고 판단하면 1~2개 정도의 키를 더 읽은 뒤 함께 반환하기도 한다.
  - 위에도 첫 번째 `SCAN` 결과 11개의 키가 반환됐다.
  - `COUNT` 옵션을 너무 크게 설정하면 출력에 오랜 시간이 걸리면서 서비스에 영향을 줄 수 있다.

<br>

`MATCH` 옵션을 사용하면 `KEYS`에서처럼 입력한 패턴에 맞는 키 값을 조회한다.  
그러나 한 번에 패턴에 매칭된 여러 개의 키가 조회되지 않는다.
```redis
> SCAN 0 match *11*
1) "19"
2) 1) "set:111"
   2) "score:251113"

> SCAN 19 match *11*
1) "0"
2) (empty array)
```
`SCAN` 커맨드에서 `MATCH` 옵션을 사용할 때에는 우선 데이터를 필터링 없이 스캔한 다음 데이터를 반환하기 직전에 필터링하는 방식으로 동작하기 때문이다.

<br>

`TYPE` 옵션을 사용하면 지정한 타입의 키만 조회할 수 있다.  
`MATCH` 옵션과 마찬가지로 반환되기 전에 필터되는 방식으로 동작한다.
```redis
> SCAN 0 TYPE zset
1) "19"
2) 1) "travel"
   2) "score:251113"

> SCAN 19 TYPE zset
1) "0"
2) 1) "mySortedSet"
```

<br>

> `SCAN`과 비슷한 커맨드로 `SSCAN`, `HSCAN`, `ZSCAN`이 있다.  
> 각각 `set`, `hash`, `sorted set`에서 아이템을 조회하기 위해 사용되는 `SMEMBERS`, `HGETALL`, `ZRANGE WITHSCORES`를 대체해서 서버에 최대한 영향을 끼치지 않고 반복해서 호출할 수 있도록 사용되는 커맨드다.

<br>

`SORT`  
```redis
SORT key [BY by-pattern] [LIMIT offset count] [GET get-pattern [GET get-pattern ...]] [ASC|DESC] [ALPHA] [STORE destination]
```
- `list`, `set`, `sorted set`에서만 사용할 수 있는 커맨드다.
- `LIMIT` 옵션을 사용하면 일부 데이터만 조회할 수 있다.
- `ASC/DESC` 옵션을 사용하면 정렬 순서를 변경할 수 있다.
- 정렬할 대상이 문자열일 경우 `ALPHA` 옵션을 사용하면 데이터를 사전 순으로 정렬해 조회할 수 있다.

```redis
127.0.0.1:6379> LPUSH mylist a
(integer) 1
127.0.0.1:6379> LPUSH mylist b
(integer) 2
127.0.0.1:6379> LPUSH mylist c
(integer) 3
127.0.0.1:6379> LPUSH mylist hello
(integer) 4
127.0.0.1:6379> SORT mylist
(error) ERR One or more scores can't be converted into double
127.0.0.1:6379> SORT mylist ALPHA
1) "a"
2) "b"
3) "c"
4) "hello"
```

<br>

`RENAME` / `RENAMENX`  
```redis
RENAME key newkey
RENAMENX key newkey
```
두 커맨드 모두 키의 이름을 변경하는 커맨드다.  
하지만 `RENAMENX` 커맨드는 오직 변경할 키가 존재하지 않을 때에만 동작한다.

```redis
127.0.0.1:6379> SET a apple
OK
127.0.0.1:6379> SET b banana
OK

// a를 aa로 변경
127.0.0.1:6379> RENAME a aa
OK
127.0.0.1:6379> GET aa
"apple"

// a에 apple 저장
127.0.0.1:6379> SET a apple
OK

// a와 aa 모두 apple
127.0.0.1:6379> GET a
"apple"
127.0.0.1:6379> GET aa
"apple"

// a가 존재하므로 변경 실패
127.0.0.1:6379> RENAMENX a b
(integer) 0
127.0.0.1:6379> get b
"banana"

// a를 b로 변경
127.0.0.1:6379> RENAME a b
OK

// b에는 apple이 들어있다.
127.0.0.1:6379> get b
"apple"
```

<br>

`COPY`
```redis
COPY source destination [DB destination-db] [REPLACE]
```
`source`에 지정된 키를 `destination` 키에 복사한다.  
`destination`에 지정한 키가 이미 있는 경우 에러가 반환된다.  
`REPLACE` 옵션을 사용하면 `destination` 키를 삭제한 뒤 값을 복사하기 때문에 에러가 발생하지 않는다.  

```redis
127.0.0.1:6379> SET B banana
OK
127.0.0.1:6379> COPY B BB
(integer) 1
127.0.0.1:6379> GET B
"banana"
127.0.0.1:6379> GET BB
"banana"
```

<br>

`TYPE`
```redis
TYPE key
```
지정한 키의 자료 구조 타입을 반환한다.

<br>

`OBJECT`
```redis
OBJECT <subcommand> [<arg> [value] [opt] ...]
```
키에 대한 상세 정보를 반환한다.  
- `subcommand` 옵션: `ENCODING`, `IDLETIME` 등이 있다.  
- 해당 키가 내부적으로 어떻게 저장됐는지, 키가 호출되지 않은 시간이 얼마나 됐는지 등을 확인할 수 있다.

<br>

### ❗️ 키의 삭제
`FLUSHALL`
```redis
FLUSHALL [ASYNC|SYNC]
```
레디스에 저장된 모든 키를 삭제한다.
- 기본적으로 `FLUSHALL`은 `SYNC`한 방식으로 동작해 모든 데이터가 삭제한 경우에만 OK를 반환한다.   
    ➡ 커맨드가 실행되는 도중에는 다른 응답을 처리할 수 없다.
- `ASYNC` 옵션을 사용하면 `flush`는 백그라운드로 실행되고, 커맨드가 수행됐을 때 존재했던 키만 삭제해서 flush되는 중 새로 생성된 키는 삭제되지 않는다. 

`lazyfree-lazy-user-flush` 옵션이 yes인 경우 `ASYNC` 옵션 없이 `FLUSHALL` 커맨드를 사용하더라도 백그라운드로 키 삭제 작업이 동작한다.

<br>

`DEL`
```redis
 DEL key [key ...]
```

<br>

`UNLINK`
```redis
UNLINK key [key ...]
```
`DEL`과 비슷하게 키와 데이터를 삭제하는 커맨드다.
- 하지만 백그라운드에서 다른 스레드에 의해 처리된다.
- 우선 키와 연결된 데이터의 연결을 끊는다.

> `set`, `sorted set`과 같이 하나의 키에 여러 개의 아이템이 저장된 자료 구조의 경우 `DEL`이 아니라 `UNLINK`를 사용해 데이터를 삭제하는 것이 좋다.


<br>

### ❗️ 키의 만료 시간
`EXPIRE`
```redis
EXPIRE key seconds [NX|XX|GT|LT]
```
키가 만료될 시간을 초 단위로 정의할 수 있다.
- `NX`: 해당 키에 만료 시간이 정의돼 있지 않을 경우에만 커맨드 수행
- `XX`: 해당 키에 만료 시간이 정의돼 있을 경우에만 커맨드 수행
- `GT`: 현재 키가 가지고 있는 만료 시간보다 새로 입력한 초가 더 클 때에만 수행
- `LT`: 현재 키가 가지고 있는 만료 시간보다 새로 입력한 초가 더 작을 때에만 수행

<br>

`EXPIREAT`
```redis
EXPIREAT key unix-time-seconds [NX|XX|GT|LT]
```
키가 특정 유닉스 타임스탬프에 만료될 수 있도록 키의 만료 시간을 직접 지정한다.

<br>

`EXPIRETIME`
```redis
EXPIRETIME key
```
키가 삭제되는 유닉스 타임스탬프를 초 단위로 반환한다.  
키가 존재하지만 만료 시간이 설정돼 있지 않은 경우에는 -1을, 키가 없을 때에는 -2를 반환한다.

<br>

`TTL`
```redis
TTL key
```
키가 몇 초 뒤에 만료되는지 반환한다.  
키가 존재하지만 만료 시간이 설정돼 있지 않은 경우에는 -1을, 키가 없을 때에는 -2를 반환한다.

<br>

> 밀리초 단위로 계산하려면 `PEXPIRE`, `PEXPIREAT`, `PEXPIRETIME`, `PTTL`을 사용하면 된다. 



<br>

**참고 자료**  
[개발자를 위한 레디스](https://product.kyobobook.co.kr/detail/S000210785682)