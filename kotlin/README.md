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