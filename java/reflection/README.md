# ❗️ 리플렉션(Reflection)
> 구체적인 클래스 타입을 알지 못해도 그 클래스의 메소드, 타입, 변수들에 접근할 수 있도록 해주는 Java API를 의미한다.  
> 리플렉션 기술을 사용하면 클래스나 메서드의 메타정보를 동적으로 획득하고, 코드도 동적으로 호출할 수 있다.

### 💻 `reflection0()`
```java
package hello.proxy.jdkdynamic;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;

@Slf4j
public class ReflectionTest {

    @Test
    void reflection0() {
        Hello target = new Hello();
        
        // 공통 로직1 시작
        log.info("start");
        String result1 = target.callA(); // 호출하는 메서드가 다름
        log.info("result={}", result1);
        // 공통 로직1 종료
        
        // 공통 로직2 시작
        log.info("start");
        String result2 = target.callB(); // 호출하는 메서드가 다름
        log.info("result={}", result2);
        // 공통 로직2 종료
        
    }

    @Slf4j
    static class Hello {
        public String callA() {
            log.info("callA");
            return "A";
        }
        
        public String callB() {
            log.info("callB");
            return "B";
        }
    }
}
```
- 공통 로직1과 공통 로직2는 호출하는 메서드만 다르고 전체 코드 흐름이 완전히 같다.
- 호출하는 메서드인 `target.callA()`, `target.callB()` 이 부분만 동적으로 처리할 수 있다면 하나의 메서드로 합칠 수 있을 것 같다.
```java
log.info("start");
String result = xxx(); //호출 대상이 다름, 동적 처리 필요
log.info("result={}", result);
```
- 이럴 때 사용하는 기술이 바로 리플렉션이다.
- 리플렉션은 클래스나 메서드의 메타정보를 사용해서 동적으로 호출하는 메서드를 변경할 수 있다.


### 💻 `reflection1()`
```java
@Test
void reflection1() throws Exception {
    // 클래스 메타정보
    // 참고로 내부 클래스는 구분을 위해 $를 사용한다.
    Class classHello = Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");

    Hello target = new Hello();
    // callA 메서드 정보
    Method methodCallA = classHello.getMethod("callA");
    Object result1 = methodCallA.invoke(target);
    log.info("result1={}", result1);

    // callA 메서드 정보
    Method methodCallB = classHello.getMethod("callB");
    Object result2 = methodCallB.invoke(target);
    log.info("result2={}", result2);
}
```
- `classHello.getMethod("callA")`
  - 해당 클래스의 `callA()` 메서드 메타정보를 획득한다.
- `methodCallA.invoke(target)`
  - 획득한 메서드 메타정보로 **실제 인스턴스의 메서드를 호출**한다.
  - `methodCallA`: `callA()`라는 메서드 메타정보
  - `target`: 실제 인스턴스
  - 따라서 해당 인스턴스의 `callA()`를 호출한다.
- 이제 하나의 메서드로 합치는 것이 가능해졌다.

### 💻 `reflection2()`
```java
@Test
void reflection2() throws Exception {
    // 클래스 정보
    Class classHello = Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");

    Hello target = new Hello();
    // callA 메서드 정보
    Method methodCallA = classHello.getMethod("callA");
    dynamicCall(methodCallA, target);

/*
    Object result1 = methodCallA.invoke(target);
    log.info("result1={}", result1);
*/


    // callB 메서드 정보
    Method methodCallB = classHello.getMethod("callB");
    dynamicCall(methodCallB, target);

/*
    Object result2 = methodCallB.invoke(target);
    log.info("result2={}", result2);
*/
}

private void dynamicCall(Method method, Object target) throws Exception {
    log.info("start");
    Object result = method.invoke(target);
    log.info("result={}", result);

}
```
- `dynamicCall(Method method, Object target)`
  - 공통 로직1, 공통 로직2를 한번에 처리할 수 있는 통합된 공통 처리 로직이다.
  - `Method method`
    - 첫 번째 파라미터는 호출할 메서드 정보가 넘어온다.
    - 이것이 핵심이다.
    - 기존에는 메서드 이름을 직접 호출했지만, 이제는 `Method`라는 메타정보를 통해서 호출할 메서드 정보가 동적으로 제공된다.
  - `Object target`
    - 실제 실행할 인스턴스 정보가 넘어온다.
    - 타입이 `Object`라는 것은 어떠한 인스턴스도 받을 수 있다는 뜻이다.
    - 물론 `method.invoke(target)`를 사용할 때 호출할 클래스와 메서드 정보가 서로 다르면 예외가 발생한다.
      - `target` 객체에 `method`로 전달한 메서드가 존재해야 한다.

### ❗️ 주의
- 리플렉션 기술은 런타임에 동작하기 때문에, 컴파일 시점에 오류를 잡을 수 없다.
  - 예를 들어, `getMethod("callA")`가 아니라 `getMethod("callZ")`로 작성해도 컴파일 오류는 발생하지 않는다.
- 가장 좋은 오류는 개발자가 즉시 확인할 수 있는 컴파일 오류이고, 가장 무서운 오류는 사용자가 직접 실행할 때 발생하는 런타임 오류다.
- 그리고 런타임 시점에 클래스를 분석하기 때문에 JVM을 최적화할 수 없기 때문에 성능 저하가 발생하므로 일반 메서드 호출보다 성능이 훨씬 떨어진다.
- 따라서 리플렉션은 일반적으로 사용하면 안 된다.

출처  
[스프링 핵심 원리 - 고급편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8)