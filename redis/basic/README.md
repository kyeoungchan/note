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

**참고 자료**  
[개발자를 위한 레디스](https://product.kyobobook.co.kr/detail/S000210785682)