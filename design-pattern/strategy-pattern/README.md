# 🖋️ 전략 패턴
> 전략 패턴(Strategy Pattern) 또는 정책 패턴(Policy Pattern)은 실행 중에 알고리즘을 선택할 수 있게 하는 행위 소프트웨어 디자인 패턴이다.  
> 전략 패턴은 특정한 계열의 알고리즘들을 정의하고 각 알고리즘을 캡슐화하며 이 알고리즘들을 해당 계열 안에서 상호 교체가 가능하게 만든다.

## 💡 예제
Movable 인터페이스를 상속받는 Bus와 Train이 있다.
```java
interface Movable {
    void move();
}

class Bus implements Movable {
    public void move() {
        System.out.println("차도로 이동합니다.");
    }
}

class Train implements Movable {
    public void move() {
        System.out.println("선로로 이동합니다.");
    }
}
```
<br>

이때 **선로로 이동하는 버스가 나와** Bus의 move()를 **OCP 원칙에 준수**하여 수정해야 한다.
=> 이동 "전략" 클래스를 생성한다.
```java
interface MovableStrategy {
    void move();
}

public class RailLoadStrategy implements MovableStrategy {
    public void move() {
        System.out.println("선로로 이동합니다.");
    }
}

public class LoadStrategy implements MovableStrategy {
    public void move() {
        System.out.println("차도로 이동합니다.");
    }
}
```
<br>

위의 전략을 통해 움직이는 Moving 객체를 생성한다.
```java
class Moving {
    private MovableStrategy movableStrategy;
    
    public void move() {
        movableStrategy.move();
    }

    public void setMovableStrategy(MovableStrategy movableStrategy) {
        this.movableStrategy = movableStrategy;
    }
}
```
<br>

따라서 클라이언트에 이와 같이 사용 가능하다.
```java
public class Client {
    public static void main(String[] args) {
        Moving train = new Moving();
        Moving bus = new Moving();
        
        /*
                기존의 기차와 버스의 이동 방식
                1) 기차 - 선로
                2) 버스 - 도로
         */
        train.setMovableStrategy(new RailLoadStrategy());
        bus.setMovableStrategy(new LoadStrategy());
        
        /*
                선로를 따라 움직이는 버스가 개발
         */
        bus.setMovableStrategy(new RailLoadStrategy());
        bus.move();
    }
}
```

> **전략패턴의 단점**  
> 간단한 경우에는 그냥 if문으로 분기처리하는 게 더 편할 수 있다.

## 💡 Comparator

```java
import java.util.Comparator;
import java.util.Collections;

public class AgeComparator implements Comparator<Person> {
    @Override
    public int compare(Person p1, Person p2) {
        return Integer.compare(p1.getAge(), p2.getAge());
    }
}

public class NameComparator implements Comparator<Person> {
    @Override
    public int compare(Person p1, Person p2) {
        return p1.getName().compareTo(p2.getName());
    }
}

// 사용 예시
List<Person> people = ...;
Collections.sort(people, new AgeComparator()); // 나이 기준으로 정렬
Collections.sort(people, new NameComparator()); // 이름 기준으로 정렬
```