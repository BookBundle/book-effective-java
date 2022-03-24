# private 생성자나 열거 타입으로 싱글턴임을 보증하라

- 싱글턴은 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.
- 무상태 객체나 설계상 유일해야 하는 시스템 컴포넌트에 사용된다.
- 클래스를 싱글턴으로 만들게 되면 테스트하기가 어려워진다.

## 싱글턴 만드는 방식

### private static final 필드 방식의 싱글턴

- 생성자는 private으로 두고 public static final 변수로 인스턴스로 초기화한다.
- 필드가 초기화될 때 생성자를 한 번만 호출하고 생성자는 private으로 되어있어 인스턴스가 하나임을 보장한다.

#### 장점

- 해당 클래스는 싱글턴임을 API에 명백히 드러난다.
- 간결하다.

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
}
```

### 정적 팩터리 방식의 싱글턴

- 필드는 private로 막고 getInstance 메소드를 만들어 호출해서 접근할 수 있도록 한다.

#### 장점

- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 점
    - 이전 방식은 싱글턴이 필요 없다면 제공하는 API 방식이 바뀌게 되지만 정적 팩터리는 바꿀 필요가 없다.
- 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다. (아이템 30)
- 정적 팩터리의 메서드  참조를 공급자(supplier)로 사용할 수 있다는 점이다. (아이템 43, 44)
    - Elvis::getInstance -> Supplier<Elvis>

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {};
    public static Elvis getInstance() { return INSTANCE; }
}
```

### 위 방식 문제점

- 직렬화 - 역직렬화, 리플렉션 시에 인스턴스가 하나임을 보장하지 못한다.

#### Reflection

```java
public class Main {
    public static void main(String[] args) throws Exception {
        final Constructor<Elvis> constructor = Elvis.class.getDeclaredConstructor();
        constructor.setAccessible(true);
        final Elvis elvis = constructor.newInstance();
        final Elvis elvis2 = constructor.newInstance();
        System.out.println(elvis == elvis2); // false
    }
}
```

- 리플렉션 문제는 생성자 안에 새로운 인스턴스를 만들려고 할 때 예외를 던져주면 해결할 수 있다.

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {
        if (INSTANCE != null) {
            throw new RuntimeException("비정상적인 접근입니다.");
        }
    }
}
```

#### Serializable

```java
public class Elvis implements Serializable {
    ...
}

public class Main {
    public static void main(String[] args) throws Exception {
        final Elvis instance = Elvis.INSTANCE;
        final Elvis elvis;
        try (ObjectOutput out = new ObjectOutputStream(new FileOutputStream("elvis.obj"))) {
            out.writeObject(instance);
        }

        try (ObjectInput in = new ObjectInputStream(new FileInputStream("elvis.obj"))) {
            elvis = (Elvis) in.readObject();
        }
    }
}

System.out.println(instance == elvis); // false
```

- 역직렬화 문제는 클래스에 readResolve 함수를 정의하고 모든 인스턴스 필드에 transient라고 선언하면 해결할 수 있다.

```java
public class Elvis implements Serializable {
    ...
    protected Object readResolve() {
        return INSTANCE;
    }
}
```

### Enum(열거) 타입 방식의 싱글턴

- 대부분의 상황에서 싱글턴임을 보장하기 위한 방법으로 가장 좋다.
- 복잡한 직렬화 상황이나 리플렉션 공격에도 제2의 인스턴스가 생기는 일을 차단한다.

```java
public enum Elvis {
    INSTANCE;
}
```