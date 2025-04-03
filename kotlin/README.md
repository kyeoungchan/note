# 💻 [Kotlin] 코틀린 문법 총정리

## ✅ 변수
1. `var`: 변수값 변경 가능
2. `val`: 선언시에만 초기화 가능(변경 불가능) -> JAVA의 **final**
```kotlin
fun main() {
    var a: Int // 자료형 선언시 -> 변수: type
    a = 123
    print(a)
}
```
```kotlin
fun main() {
    val b: Int = 1232
    b = 3 // 중간에 값을 못 바꾸기 때문에 에러
    print(b)
}
```
> ?: 변수 값이 `null`일 수 있다는 것을 표시  
> ?를 표시하지 않으면 선인 시 `null`이 될 수 없다.
```kotlin
fun main() {
    var a: Int? = null
    print(a)
}
// 출력: null
```

<br>

## ✅ 형변환
코틀린에서는 `to자료형()`를 통해 형변환 가능  
코틀린은 암시적 형변환을 지원하지 않는다.
```kotlin
fun main() {
    var a: Int = 123
    var b: String = a.toString()
    var c: Int = b.toInt()
    print(b)
}
```

<br>

## ✅ 배열
```kotlin
fun main() {
    // int형으로 1 2 3 4 배열 생성
    var intArr: Array<Int> = arryOf(1, 2, 3, 4)
    
    // type 생략 가능
    var intArr2 = arrayOfNulls<Int>(5)
    
    // Any는 데이터 타입의 최상위(어느 데이터든 다 들어갈 수 있다.)
    var anyArr: Array<Any> = arrayOf(1, "charles", 3.2, 4)
    
    print(intArr[0]) // 1
    print(intArr2[1]) // null
    print(anyArr[1]) // charles
}
```

<br>

## ✅ 함수
기본형 함수
```kotlin
fun main() {
    print(add(1, 2, 3)) // 6
}

// 함수의 기본형 fun 함수이름(매개변수: type): 리턴타입
fun add(a: Int, b: Int, c: Int): Int {
    return a + b + c
}
```
단일 표현식 함수
```kotlin
fun main() {
    print(add(1, 2, 3)) // 6
}

// int a, b, c를 더하므로 반환형 타입이 int라 추론 가능
fun add(a: Int, b: Int, c: Int) = a + b + c
```

<br>

## ✅ 조건문
if: 다른 언어랑 똑같음
```kotlin
fun main() {
    var a = 7
    if (a > 6) {
        print(a)
    } else {
        print("exit")
    } 
}
// 출력 7
```
is: 데이터 타입 비교
```kotlin
fun main() {
    var a: Any = 1
    
    if (a is Int) {
        print("int")
    }
    if (a is String) {
        print("string")
    }
}
// 출력 int
```
when: switch문이랑 비슷한 기능
```kotlin
// 동작 실행
fun main() {
    exWhen(2)
}

fun exWhen(a: Any) {
    when (a) {
        1 -> print(a)
        "charles" -> print(a)
        else -> print(a)
    }
}
// 출력: 2
```
```kotlin
// 값 설정
fun main() {
    exWhen(2)
}

fun exWhen(a: Any) {
    var b =
        when (a) {
            1 -> a
            "charles" -> a
            else -> a
        }
    print(b)
}
// 출력: 2
```

<br>

## ✅ 반복문
while: 다른 언어랑 똑같음
```kotlin
fun main() {
    var i: Int = 0
    while (i < 3) {
        print(i)
        i++
    }
}
```
for
```kotlin
fun main() {
    for (i in 0..3) {
        print(i)
        print(" ")
    }
    
    println()

    for (i in 3 downTo 0) {
        print(i)
        print(" ")
    }

    println()

    for (i in 'a'..'e') {
        print(i)
        print(" ")
    }
}
/*
결과
0 1 2 3
3 2 1 0
a b c d e
 */
```

<br>

## ✅ 흐름 제어
break, continue: 다른 언어랑 똑같음
```kotlin
fun main() {
    for (i in 0..5) {
        if (i == 2) {
            break
        }
        println(i)
    }

    println()

    for (i in 0..5) {
        if (i == 2) {
            continue
        }
        println(i)
    }
}
/*
출력
0
1

0
1
3
4
5
 */
```
@label: 반복문에 `라벨이름@`을 달고 break, continue 문에 `@라벨이름`을 달면 break, continue 가 라벨이름으로 가 실행
```kotlin
fun main() {
    charles@ for (i in 0..10) {
        for (j in 0..10) {
            if (i == 0 && j == 3) {
                break@charles
            }
            println("$i, $j")
        }
    }
}
/*
출력
0, 0
0, 1
0, 2
 */
```

<br>

## ✅ 클래스
### 기본형
클래스 형태: `class 클래스이름(속성 이름: type`
Java랑 다르게 코틀린은 생성자를 따로 만들어주지 않아도 된다.
객체를 생성할 때 속성 따라 값을 입력해주면 된다.

```kotlin
fun main() {
    var a = Person("charles", 23)
    println("${a.birth} ${a.name}")
    a.introduce()
}

class Person(var name: String, val birth: Int) {
    fun introduce() {
        println("$name $birth")
    }
}
/*
출력
23 charles
charles 23
*/
```

### init
보통 Java에서는 생성자를 만들면 구문을 수행하는 기능을 넣을 수 있다.
하지만 위 코드처럼 생성자를 만들면 생성자를 실행하면서 구문을 넣을 수 없다.
`init`을 사용하여 생성자를 출력했을 시 구문이 실행되게 할 수 있다.
`init`은 여러 개 둘 수 있다.
```kotlin
fun main() {
    var a = Person("charles", 23)
} 

class Person(var name: String, val birth: Int) {
    init {
        println("${this.name} ${this.birth}")
    }

    init {
        println(1)
    }
}
/*
출력
charles 23
1
 */
```

### constructor(보조 생성자)
constructor 오버로딩 생성자를 만들 수 있다.
```kotlin
fun main() {
    var a = Person("charles")
}

class Person(var name: String, val birth: Int) {
    // this를 사용해 기본 생성자로 값을 넘겨줘야한다.
    constructor(name:String) : this(name, 23)

    init {
        println("${this.name} ${this.birth}")
    }
}
/*
출력
charles 23
 */
```

### 상속
클래스를 상속할 때 부모 클래스는 `open`이라는 키워드가 있어야 상속이 가능하다.
또한 서브 클래스의 모든 속성들은 부모 클래스의 속성과 이름이 같으면 안 된다.
함수를 override 할 때는 `override` 키워드 사용
함수를 override 할 때도 부모 클래스의 함수에 `open` 키워드를 사용해야 한다.

```kotlin
fun main() {
    var dog = Dog("a", 12)
    dog.introduce1()
    dog.introduce()
}

open class Animal(var name: String, var age: Int, var type: String) {
    open fun introduce() {
        println("${this.name} ${this.age} ${this.type}")
    }
}

class Dog(var name1: String, var age1: Int) : Animal(name1, age1, "강아지") {
    fun introduce1() {
        super.introduce()
    }

    override fun introduce() {
        println("override")
    }
}
/*
출력
a 12 강아지
override
 */
```

### 추상클래스
Java 랑 똑같다.

```kotlin
fun main() {
    var a = Rabbit()
    a.eat()
    a.sniff()
}

abstract class Animal {
    abstract fun eat()
    fun sniff() {
        println("킁킁")
    }
}

class Rabbit : Animal() {
    override fun eat() {
        println("carrot eating")
    }
}
/*
출력
carrot eating
킁킁
 */
```

### 인터페이스
코틀린에서 인터페이스는 Java 인터페이스와는 다르게 추상 함수만 있는 게 아니라 속성과 일반 함수도 선언할 수 있다.  
구현부가 있는 함수 -> open 함수로 간주
구현부가 없는 함수 -> abstract 함수로 간주

```kotlin
fun main() {
    var a = Dog(1)
    a.eat()
    a.run()
}

interface Runner {
    fun run()
}

interface Eater {
    fun eat() {
        println("food eating")
    }
}

class Dog(var a: Int) : Runner, Eater {
    override fun run() {
        println("run")
    }
    
    override fun eat() {
        println("dog food eating")
    }
}
/*
출력
dog food eating
run
 */
```

<br>

## ✅ 접근제한자
Java 와 쓰는 방법은 동일하다.  
단, `default`는 public
- public: 클래스 외부에서 접근 가능
- private: 클래스 내부에서만 접근 가능
- protected: 클래스 자신과 상속받은 클래스 접근 가능

<br>

## ✅ 고차함수/람다함수
고차함수: 함수를 클래스에서 만들어낸 인스턴스처럼 취급하는 방법  
-> 함수를 파라미터로 넘겨줄 수 있음  
-> 결과값으로도 반환받을 수 있음
```kotlin
fun main() {
    b(::a) // 함수를 넘겨줄 때 :: 를 붙인다.
    
    // 람다 함수로 작성된 c
    // (입력 타입) -> 반환 타입 = {변수이름:입력타입 -> 구문}
    // 아래 두 개의 c, d처럼 타입 생략 가능
    var c: (String) -> Unit = {s -> pringln(s)}
    var d = {s:String -> println(s)}
    var e = {s:String -> s} // s를 반환
    var f = {
        println("abcd")
    }
    var g: (String) -> Unit = {println(it)} // 인자가 하나일 경우 it 키워드 사용

    c("abc")
    d("efg")
    println(e("zzz"))
    f()
    g("ggg")
}

fun a(str:String) : String {
    return str
}

fun b(funs: (String) -> String) {
    // 가져온 함수에 인자를 넣어 실행
    println(funs("bbbb"))
}
/*
결과
bbbb
abc
efg
zzz
abcd
ggg
 */
```

<br>

## ✅ 스코프 함수
함수형 언어를 좀더 편리하게 사용할 수 있도록 하는 기본 함수
apply, run, with, also, let 이 있다.
### apply
인스턴스의 값을 람다함수를 사용해 변경할 수 있는 함수 그리고 변경된 객체를 반환  
장점: 코드가 깔끔해짐
```kotlin
fun main() {
    var a = Book("a", 20000)
    // apply 스코프람다함수를 통해 a 객체의 속성과 함수 변경 및 사용 가능
    a.apply {
        name = "apply $name"
        // 2000 감소
        dc()
    }
    a.printName()
}

class Book(var name: String, var price: Int) {
    fun dc() {
        price -= 2000
    }
    
    fun printName() {
        println("$name $price")
    }
}
/*
출력
apply a 18000
 */
```

### run
apply랑 똑같은 기능이지만 마지막 구문에 있는 값을 반환해주는 차이가 있다.
```kotlin
fun main() {
    var a = Book("a", 20000)
    var b = a.run {
        name = "apply $name"
        dc()
        "retun B" // 반환
    }
    print(b)
}

class Book(var name: String, var price: Int) {
    fun dc() {
        price -= 2000
    }
    
    fun printName() {
        println("$name $price")
    }
}
// 출력: return B
```

### with
run이랑 똑같지만 사용법만 다름  
`a.run -> with(a)`
```kotlin
fun main() {
    var a = Book("a", 2000)
    var b = with(a) {
        name = "apply $name"
        dc()
        "return B"
    }
    print(b)
}

class Book(var name:String, var price:Int) {
    fun dc() {
        price -= 2000
    }
    
    fun printName() {
        println("$name $price")
    }
}

// 출력: return B
```

### let/also
`apply = also`  
`run = let`  
기능은 위처럼 같다.  
하지만 also, let의 공통된 차이점은 `it` 키워드를 사용해 객체 변수를 참조한다는 것이다.  
이유는 같은 이름 변수로 혼동이 올 수 있기 때문이다.
```kotlin
fun main() {
    var price = 5000
    var a = Book("a", 20000)
    a.run {
        // 20000원이 출력되어야하지만 main문의 price가 스코프 우선순위가 높아 5000원 출력
        println(price)
    }
}

class Book(var name:String, var price:Int) {
    
}
// 출력: 5000
```
let을 사용할 경우(also도 동일)
```kotlin
fun main() {
    var price = 5000
    var a = Book("a", 20000)
    a.let {
        println(it.price)
    }
}

class Book(var name:String, var price:Int) {
    
}

// 출력: 20000
```

### Object
객체가 하나만 필요해서 사용하는 경우에 쓰는 키워드 -> 싱글톤 디자인 패턴

```kotlin
fun main() {
    Counter.countUp()
    println(Counter.count)
    Counter.clear()
    println(Counter.count)
}

object Counter {
    var count = 0
    fun countUp() {
        count++
    }
    fun clear() {
        count = 0
    }
}
/*
출력
1
0
 */
```
class 안에도 object를 만들 수 있는데, 기존 Java에서의 `static`과 비슷하다고 생각하면 된다.  
키워드는 `companion object`이다.
```kotlin
fun main() {
    var a = Food()
    var b = Food()
    a.up() // 공용 변수 증가
    b.up() // 공용 변수 증가
    println("${Food.total}")
}

class Food() {
    companion object {
        var total = 0
    }
    fun up() {
        total++
    }
}
// 출력: 2
```

<br>

## ✅ 옵저버(Observer) 패턴
`listener.callback`이라고 부른다.  
간단히 말하면 어떠한 이벤트가 발생을 감지해 이벤트 발생시 기능이 호출되도록 하는 패턴이다.
예제만 보고 해석하자.

```kotlin
fun main() {
    EventPrinter().start()
}

interface EventListener {
    fun onEvent(count: Int)
}

class Counter(var listener: EventListener) {
    fun count() {
        for (i in 0..20) {
            if (i % 5 == 0) {
                listener.onEvent(i)
            }
        }
    }
}

class EventPrinter : EventListener {
    // 이벤트 함수
    override fun onEvent(count: Int) {
        println(count)
    }
    fun start() {
        var count = Counter(this)
        count.count()
    }
}
/*
출럭
0
5
10
15
20
 */
```
```kotlin
fun main() {
    EventPrinter().start()
}

interface EventListener {
    fun onEvent(count: Int)
}

class Counter(var listener: EventListener) {
    fun count() {
        for (i in 0..20) {
            if (i % 5 == 0) {
                listener.onEvent(i)
            }
        }
    }
}

class EventPrinter {
    fun start() {
        // 리스너를 익명클래스로 정의
        Counter(object: EventListener {
            override fun onEvent(count: Int) {
                println(cout)
            }
        }).count()
    }
}
/*
출력
0
5
10
15
20
 */
```

<br>

## ✅ 다형성 as
`as`는 클래스를 casting 하는 역할을 한다.

```kotlin
fun main() {
    var a = Drink()
    a.drink()
    
    var b: Drink = Coke()
    b.drink()

    if (b is Coke) { // if문안에서 일시적으로 캐스팅
        b.washD()
    }
    
    var c = b as Coke // Coke로 캐스팅된 c. 동시에 b도 캐스팅된다.
    c.washD()
    b.washD()
}

open class Drink {
    var name = "음료"
    open fun drink() {
        println("${name}을 마십니다.")
    }
}

class Coke : Drink() {
    var type = "콜라"
    override fun drink() {
        println("${type}을 마십니다.")
    }
    fun washD() {
        println("${type}으로 설거지합니다.")
    }
}
/*
출력
음료을 마십니다.
콜라을 마십니다.
콜라으로 설거지합니다.
콜라으로 설거지합니다.
콜라으로 설거지합니다.
 */
```

<br>

## ✅ Generic

```kotlin
fun main() {
    UsingGeneric(A()).doShout()
    UsingGeneric(B()).doShout()

    funGeneric(A())
    funGeneric(B())
}

fun <T : A> funGeneric(t: T) {
    t.shout()
}

open class A {
    open fun shout() {
        println("A shout")
    }
}

class B : A() {
    override fun shout() {
        println("B shout")
    }
}

class UsingGeneric<T : A>(val t: T) {
    fun doShout() {
        t.shout()
    }
}
/*
출력
A shout
B shout
A shout
B shout
 */
```

<br>

## ✅ Collection List
리스트에는 `listOf`, `mutableListOf`가 있다.
- `listOf`는 생성시 넣은 객체를 대체, 추가, 삭제 못함
- `mutableListOf`는 대체, 추가, 삭제 가능(add, sort, shuffle) 등 함수 지원
```kotlin
fun main() {
    val a = listOf<Int>(1, 2, 3)
    val b = mutableListOf<Int>()

    b.add(1)
    b.add(2)
    b.add(3)
    b.add(2, 6) // 2번 인덱스에 6 추가
    println(a)
    println(b)
}
/*
출력
[1, 2, 3]
[1, 2, 6, 3]
 */
```

<br>

## ✅ String function
```kotlin
fun main() {
    val test = "a.b.c.d"
    println(test.length)

    var spl = test.split(".")
    println(spl)
    
    // 배열 값들을 string으로 합쳐줌
    println(spl.joinToString())
    println(spl.joinToString("-"))

    println(test.subString(0..2))
    
    // 시작단어 boolean
    println(test.startWith("a."))
    // 끝단어 boolean
    println(test.endsWith(".d"))
    // 포함 단어 boolean
    println(test.contains("b.c"))
}
/*
출력
7
[a, b, c, d]
a, b, c, d
a-b-c-d
a.b
true
true
true
 */
```

<br>

## ✅ null 처리
Java에서는 보통 `if (변수 == null)` 형식으로 null 값을 처리한다.  
코틀린은 `?` `?:` `!!`을 통해 null 값을 처리한다.
```kotlin
fun main() {
    val a = listOf<String?>("charles", null, "wkc")
    val c = mutableListOf<String>()
    val e = mutableListOf<String>()
    val f = mutableListOf<String>()

    for (b in a) {
        // b가 null이면 .let 구문을 실행하지 않는다.
        b?.let { c.add(it) }
        
        // b가 null이면 default 값이 대신 추가된다.
        e.add(b?:"default")
    }

    println(c)
    println(e)

    for (b in a) {
        // b가 null인 것을 의도적으로 방치 -> 오류 발생
        f.add(b!!)
    }
    println(f)
}
/*
출력
[charles, wkc]
[charles, default, wkc]
Exception in thread "main" kotlin.KotlinNullPointerException
	at AppKt.main(app.kt:15)
	at AppKt.main(app.kt)
 */
```

<br>

## ✅ infix
함수를 연산자처럼 사용 가능
```kotlin
fun main() {
    println("charles" strSum 1)
    println("charles".strSum(2))

    println(3 mul 4)
    println(3.mul(4))
}

// infix fun (this에 해당되는 타입).함수이름(인자이름: 타입) : 반환 = 구문
infix fun String.strSum(x: Int): String = this + x
infix fun Int.mul(x: Int): Int = this * x
/*
출력
charles1
charles2
12
12
 */
```
> 참고: 클래스 안에 infix 함수를 적용할 경우 `this`는 클래스 객체 자신이므로 함수 이름 왼쪽에 클래스 이름은 쓰지 않는다.  
> ex) `infix fun mul(x: Int): Int = this * x`

<br>

## ✅ Data 클래스
클래스에 `has()`, `equals()`, `toString()`, `copy()`, `componentX()`를 자동으로 구현해주는 클래스
```kotlin
fun main() {
    val va = a("a", 123)

    println(va == a("a", 123))
    println(va)

    val vb = b("a", 123)

    println(vb == b("a", 123))
    println(vb)
    
    // 복사
    println(vb.copy())
    println(vb.copy(name = "b"))
    println(vb.copy(id = 234))
    
    val list = listOf<b> (
        b("a", 123),
        b("b", 234), 
        b("c", 345)
    )
    // 자동으로 변수의 위치를 판별해 a, b에 값을 넣어준다.
    for ((a, b) in list) {
        println("$a $b")
    }
}

class a(val name: String, val id: Int)
data class b(val name: String, val id: Int)
/*
출력
false
a@65ab7765
true
b(name=a, id=123)
b(name=a, id=123)
b(name=b, id=123)
b(name=a, id=234)
a 123
b 234
c 345
 */
```

<br>

## ✅ enum 클래스
enum처럼 사용하는 클래스다.
```kotlin
fun main() {
    var a = St.A
    println(a)
    println(a.msg)
    println(a.isA())
}

enum class St(val msg: String) {
    A("a"),
    B("b"),
    C("c");
    
    fun isA() = this == St.A
}
/*
출력
A
a
true
 */
```

<br>

## ✅ Map 컬렉션
```kotlin
fun main() {
    var a: MutableMap<Int, String> = mutableMapOf()
    a[1] = "a"
    b[2] = "b"

    for (i in a) {
        println("${i.key} ${i.value}")
    }
    a.remove(1)

    println(a)
    println(a[2])
}
/*
출력
1 a
2 b
{2=b}
b
 */
```

<br>

## ✅ lateinit
기본 자료형을 제외(String 가능)하고 객체 생성시 초기화를 하지 않고 변수만 선언할 수 있게 하는 키워드
```kotlin
fun main() {
    var t = a()
    println(t.test())
    t.a = "charles"
    println(t.test())
}

class a {
    lateinit var a: String
    
    fun test(): String {
        // a가 초기화됐는지 체크
        if (::a.isInitialized) {
            return a
        } else {
            return "null"
        }
    }
}
/*
출력
null
charles 
 */
```

<br>

## ✅ lazy
변수의 초기화 시점을 사용할 때 초기화하게 해주는 키워드
```kotlin
fun main() {
    val num: Int by lazy {
        println("초기화")
        7
    }
    println("start")
    println(num)
    println(num)
}
/*
출력
start
초기화
7
7
 */
```