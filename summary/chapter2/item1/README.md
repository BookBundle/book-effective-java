# 생성자 대신 정적 팩터리 메서드를 고려하라

## 정적 팩터리 메서드 예제

- boolean 값을 받아 Boolean 객체 참조로 반환해준다.

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

## 생성자보다 좋은 점 5가지

### 첫 번째, 이름을 가질 수 있다.

- 매개변수와 생성자 자체로는 반환될 객체의 특성을 제대로 나타내지 못한다.
- 아래 예제는 값이 소수인 BigInteger를 반환하는 생성자와 정적 팩터리 메서드가 있다.
- 아래쪽이 의미를 파악하는데 더 명확한 것을 알 수 있다.

```java
BigInteger(int, int, Random)
BigInteger.probablePrime(int, int, Random)
```

- 하나의 시그니처로 하나의 생성자만 만들 수 있는 것을 해결

```java
// 생성자
public class User {
    private String id;
    private String pw;

    public User(String id) { // 에러
        this.id = id;
    }
    public User(String pw) { // 에러
        this.pw = pw;
    }
}

// 정적 팩터리 메서드
public class User {
    private String id;
    private String pw;

    public static User withId(String id) {
        User user = new User();
        user.id = id;
        return user;
    }
    public static User withPw(String pw) {
        User user = new User();
        user.pw = pw;
        return user;
    }
}
```

### 두 번째, 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.

- 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용 가능
- (특히 생성 비용이 큰) 객체가 자주 요청되는 상황이라면 성능을 올릴 수 있다.

```java
public class User {
    private static final User CREATED_USER = new User();
    
    public static User getInstance() {
        return CREATED_USER;
    }
}
```

### 세 번째, 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

- 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 `엄청난 유연성`을 선물한다.

```java
class Super {
    public static Super foo() {
        return new Sub();
    }
}

class Sub extends Super {}
```

### 네 번째, 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

- RegularEnumSet, JumboEnumSet은 EnumSet을 상속받고 있다.
- noneOf는 universe의 크기에 따라 다란 객체를 반환하는 것을 볼 수 있다.

```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
    Enum<?>[] universe = getUniverse(elementType);
    if (universe == null)
        throw new ClassCastException(elementType + " not an enum");

    if (universe.length <= 64)
        return new RegularEnumSet<>(elementType, universe);
    else
        return new JumboEnumSet<>(elementType, universe);
}
```

### 다섯 번째, 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

- Parent에 getInstance 메소드를 작성하는 시점에서 반환할 객체의 클래스가 존재하지 않아도 된다.
- 리플렉션을 이용하여 당장 존재하지 않더라도 구현체를 만들어낼 수 있다.

```java
public interface Parent {
    static Parent getInstance(String path) throws Exception {
        Parent parent = null;
        try {
            Class<?> child = Class.forName(path);
            parent = (Parent) child.newInstance();

        } catch (Exception e) {
            throw new Exception();
        }
        return parent;
    }

    String getName();
}

public class Child implements Parent{
    @Override
    public String getName() {
        return "child";
    }
}

public class Main {
    public static void main(String[] args) throws Exception {
        final Parent child = Parent.getInstance("chapter2.item1.Child");
        System.out.println(child.getName());
    }
}
```

## 단점 두 가지

### 첫 번째, 상속을 하면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

### 두 번째, 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

- 생성자처럼 API 설명에 명확히 드러나지 않아 인스턴스화 할 방법을 직접 찾아야 함
- API 문서를 잘 써놓거나 메서드 이름을 잘 알려진 규약을 따라 짓는 식으로 문제를 완화

## 명명 규칙

|명명 규칙|설명|예제|
|:-:|:-:|:-:|
|from|매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드|Date.from(instant)|
|of|여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드|EnumSet.of(JACK, QUEEN, KING)|
|valueOf|from과 of 더 자세한 버전|BigInteger.valueOf(Integer.MAX_VALUE)|
|instance, getIstance|(매개변수를 받는다면)매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다.|StackWalker.getInstance(options)|
|create, newInstance|instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.|Array.newInstance(...)|
|getType|getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. Type은 팩터리 메서드가 반환할 객체의 타입|FileStore fs = Files.getFileStore(path)|
|newType|newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. Type은 팩터리 메서드가 반환할 객체의 타입|BufferedReader br = Files.newBufferedReader(path)|
|type|getType과 newType의 간결한 버전|List<...> list = Collections.list(...)|