# 💻 레디스에서 키를 관리하는 법
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
- 다음 `SCAN` 커맨드를 사용할 때 인수로 사용해야하는 커서의 위치가 첫 번째로 반환된다.
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
- `ASYNC` 옵션을 사용하면 `flush`는 백그라운드로 실행되고, 커맨드가 수행됐을 때 존재했던 키만 삭제해서 `flush`되는 중 새로 생성된 키는 삭제되지 않는다.

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