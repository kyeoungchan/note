# 💻 IoC, DI, DIP
> 결론부터 얘기하면, IoC, DI, DIP는 모두 다른 개념이다.  
> 각자 목적과 요구하는 바가 다르다.  
> 하지만 서로를 배척하는 개념은 아니다. 오히려 함께했을 때 강한 시너지가 생긴다.

## ✅ Inversion of Control(IoC, 제어의 역전)
> don't call me, I'll call you - [Hollywood principle](https://en.wiktionary.org/wiki/Hollywood_principle)  

좀더 개발 친화적인 용어로 풀어서 설명하자면, 다음과 같이 표현할 수 있다.
> IoC란 코드의 흐름을 제어하는 주체가 바뀌는 것이다.

<br>

안드로이드에서 IoC가 적용된 케이스를 살펴보자.
```java
public class MyActivity extends AppCompatActivity {
    @Override
    protected void onResume() {
        super.onResume();
        // do something
    }
    
    @Override
    protected void onPause() {
        // do something
        super.onPause();
    }
}
```
Activity 코드를 작성하는 입장에서 바라보자면, 생명주기 메서드가 호출되었을 때의 동작만 정의하고, 언제 생명주기 메서드를 호출할지는 신경쓰지 않고 있다.  
즉, Activity의 메인 흐름 제어권은 나의 코드가 아니라 안드로이드 플랫폼에서 쥐고 있다.

<br>

'프레임워크와 라이브러리의 차이는 무엇인가?'에 대해 IoC 관점으로 설명이 가능하다.

| 구분    | 내용                                       |
|-------|------------------------------------------|
| 프레임워크 | 프레임워크가 나의 코드를 실행한다.<br>즉, 제어권은 프레임워크에 있다. |
|라이브러리| 내 코드가 라이브러리를 이용한다.<br>즉, 제어권은 프레임워크에 있다. |

## ✅ Dependency Inversion Principle(DIP, 의존관계 역전 법칙)
> High-level modules should not depend on low-level modules. Both should depend on abstractions(e.g. interfaces).  
> Abstractions should not depend on details. Details(concrete implementations) should depend on abstractions


<br>

> DIP is about the level of the abstraction in the messages sent from your code to the thing it is calling

DIP가 주장하는 바의 핵심은 추상화에 의존하라는 것이다.


추상화가 아닌 구체클래스에 의존한 경우
```java
public class FileLoader {
    private TextFileParser textFileParser;
    
    public FileLoader() {
        // TextFile이 아닌 csv 파일을 파싱해야할 경우 필연적으로 코드의 변경이 발생
        this.textFileParser = new TextFileParser;
    }

    public File parseFile(String serializedFile) {
        return textFileParser.parse(serializedFile);
    }
}
```
위와 같이 `TextFileParser`에 변경이 발생했을 때 이를 의존하는 `FileLoader` 역시 변경이 발생하게 된다. 또한 `FileLoader`는 `TextFile`외에 다른 `File`을 파싱하기 위해서는 아예 `Parser` 클래스를 변경해야 한다. 변경에 유연하지 않은 구조다.

추상화에 의존할 경우
```java
public interface FileParser {
    File parse(String serializedFile);
}

public class TextFileParser implements FileParser {
    @Override
    public File parse(String serializedFile) {
        // Do something
        return File();
    }
}

public class FileLoader {
    private FileParser fileParser;

    public FileLoader(FileParser fileParser) {
        this.fileParser = fileParser;
    }

    public File parseFile(String serializedFile) {
        return fileParser.parse(serializedFile);
    }
}
```
`FileLoader`는 `FileParser` 인터페이스에 의존하기에 `FileParser`의 구현체인 `TextFileParser` 변경이 발생하더라도 영향을 받지 않는다. 또한 `FileParser` 인터페이스를 구현한 구현체라면 무엇이든 `FileLoader`에서 이용이 가능하다. 즉, 다형성을 활용하여 변경에 유연한 구조가 된다.j

## ✅ Dependency Injection(DI, 의존성 주입)
[DI에 대해 따로 정리한 페이지가 있다.](https://github.com/kyeoungchan/note/tree/main/spring/dependency-injection)
> DI is about how one object acquires a dependency

DI는 필요로 하는 객체를 스스로 생성하는 것이 아닌 외부로부터 주입받는 기법을 의미한다. 마틴 파울러의 글에 따르면 3가지 타입으로 정의할 수 있다.

### 💡 Constructor Injection
생성자를 통해 주입하는 방식이다. 인스턴스가 생성되었을 때 의존성이 존재하는 것이 보장되기 때문에 의존성이 존재여부가 보자오디고 의존성을 immutable하게 정의할 수 있다. [스프링](https://github.com/kyeoungchan/note/tree/main/spring)에서도 해당 방식을 권장한다.
```java
public class FileLoader {
    private FileParser fileParser;

    public FileLoader(FileParser fileParser) {
        this.fileParser = fileParser;
    }
}
```

### 💡 Setter Injection
Setter 메서드를 이용하여 주입하는 방식이다. 해당 방식은 Constructor Injection보다 좀더 주의를 요한다. 주입받는 의존성의 기본값을 정의할 수 있지 않다면 null 값이 존재항 수 있는 이슈가 있기 때문이다. 의존성이 다시 주입되어야할 경우 유용하게 사용된다고 하나 웬만하면 모두 Constructor Injection으로 해결할 수 있다.

```java
public class FileLoader {
    private FileParser fileParser;

    public void setFileParser(FileParser fileParser) {
        this.fileParser = fileParser;
    }
}
```

### 💡 Interface Injection
Interface로 주입받는 메서드를 정의한다.


<br>

## ✅ 각 개념간의 관계
### 💡 IoC와 DI
종종 IoC와 DI는 interchangeably한 것, 더 나아가 아예 같은 것처럼 표현하는 글이 보이곤 하는데 이는 잘못된 해석이라고 생각한다. Dependency Injection은 IoC 개념이 적용된 결과물 중 하나일 뿐이다. 의존성을 주입한다는 것을 IoC 적인 행위로 바라볼 수는 있지만 IoC가 곧 의존성 주입이라고 보기는 어렵기 때문이다.

### 💡 DIP와 DI
단어가 비슷해보이는 DIP와 DI 역시 같은 개념으로 오해하기 쉽다. 하지만 마찬가지로 DI는 DIP를 구현하는 기법 중 하나일 뿐 서로 같은 개념이 아니다. 위 DIP 예제 코드에서도 DI가 사용되었다.

DIP에 대한 이해가 부족했을 때, 아래와 같은 코드도 DIP를 만족하는 것이라고 생각했었다.
```java
public interface FileParser {
    File parse(String serializedFile);
}

public class TextFileParser implements FileParser {
    @Override
    public File parse(String serializedFile) {
        // Do something
        return File();
    }
}

public class FileLoader {
    private FileParser fileParser;
    
    public FileLoader() {
        this.fileParser = new TextFileParser();
    }

    public File parseFile(String serializedFile) {
        return fileParser.parse(serializedFile);
    }
}
```

하지만 해당 코드에서는 `FileParser`를 다른 구현체로 바꿀 수 없다. 사실상 타입만 인터페이스로 했을 뿐 다형성의 이점을 전혀 살리지 못하는 코드이며 DIP를 만족한다고 보기 어렵다.

또 다른 예시를 살펴보자. 이 코드도 조금 아쉬운 점이 있다.
```java
public class FileLoader {
    private TextFileParser textFileParser;

    public FileLoader(TextFileParser textFileParser) {
        this.textFileParser = textFileParser;
    }

    public File parseFile(String serializedFile) {
        return textFileParser.parse(serializedFile);
    }
}
```
해당 코드는 `TextFileParser`를 주입받으므로 DI가 이루어졌다고 볼 수 있다. 따라서 `TextFileParser`의 생성자에 변경이 생기더라도 `FileLoader`에 전파되지 않는 것은 긍정적인 부분이다. 하지만 DIP는 지켜지지 않았다. 구체 클래스에 의존하고 있으므로 다른 `FileParser`로 교체하는 것은 불가능하며 `TextFileParser`의 변경에 `FileLoader`가 영향을 받게 된다.

위 예시들이 시사하는 바는 **DIP와 DI가 서로 조합되었을 때 시너지를 발휘한다는 것**이다. 그래서 보통 한쪽 개념의 예시를 들 때 다른 쪽 개념이 같이 적용되어있기 때문에 두 개념을 같다고 이해할 법도 하다.

<br>

**출처**
- [[Study]IoC, DI, DIP 개념 잡기](https://vagabond95.me/posts/about-ioc-dip-di/)