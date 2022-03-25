# 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

- 대부분의 클래스는 하나 이상의 자원에 의존한다.
- 맞춤법 검사기와 같은 클래스는 정적 유틸리티 클래스로 구현을 많이 한다.

## 잘못 사용한 예

- 아래 두 방식은 사전 하나만 사용한다고 가정하였다.
- 사전이 언어별로 따로 있거나 특수 어휘용 사전이 별도로 있다면 사전 하나로 모든 것을 대응할 수 없다.

### 정적 유틸리티를 잘못 사용한 예

```java
public class SpellChecker {
    private static final Lexicon dictionary = ...; // 좋지 않음

    private SpellChecker() {}

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

### 싱글턴을 잘못 사용한 예

```java
public class SpellChecker {
    private final Lexicon dictionary = ...; // 좋지 않음

    private SpellChecker() {}
    public static SpellChecker INSTANCE = new SpellChecker();

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

## 메소드로 해결 방법

- final 키워드를 제거하고 사전을 바꾸는 메서드를 추가한다.
- 하지만 이 방식은 멀티스레드 환경에서 쓸 수 없고 어색하고 오류 내기가 쉽다.
- **사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.**

```java
public class SpellChecker {
    private Lexicon dictionary = ...;

    private SpellChecker() {}

    public changeDictionary(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

## 의존성 주입 (지향)

- 인스턴스 생성할 때 생성자에 필요한 자원을 넘겨주는 방식을 이용한다.
- 자원이 몇 개든 의존 관계가 어떻든 상관없고 불변을 보장하여 의존 객체들을 안심하고 공유할 수 있다.
- 유연하고 테스트하기 쉽다.

```java
public class SpellChecker { 
    private final Lexicon dictionary;

    public SpellChecker(final Lexicon dictionary) {
        this.dictionary = dictionary;
    }

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

## 쓸만한 변형

- 생성자에 자원 팩터리를 넘겨주는 방식이 있다.
    - 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말함
    - `Supplier<T>`
- 한정적 와일드카드를 이용해서 타입을 제한할 수도 있다.

```java
public interface Tile {}
public class StoneTile implements Tile{}
public class StonewareTile implements Tile{}

public class Mosaic {

    private final List<Tile> tiles;

    public Mosaic(final Supplier<? extends Tile> tileFactory) {
        tiles = new ArrayList<>();
        tiles.add(tileFactory.get());
        tiles.add(tileFactory.get());
        tiles.add(tileFactory.get());
        tiles.add(tileFactory.get());
    }
}

public class Main {

    public static void main(String[] args) {
        final Mosaic mosaic = new Mosaic(StoneTile::new);
    }
}
```

## 단점

- 의존 객체 주입이 유연성과 테스트 용이성을 개선하지만, 의존성이 많은 큰 프로젝트에선 코드가 어지러울 수 있다.
- `Guice`, `Dagger`, `Spring` 같은 객체 주입 프레임워크의 도움으로 이를 해소할 수 있다.

## 핵심 정리

- 클래스가 내부적으로 하나 이상의 자원에 의존하고, 자원이 클래스 동작에 영향을 준다면 싱글턴, 정적 유틸리티 클래스로 사용하지 않는 것이 좋다.
- 클래스에서 직접 생성하고 변경해서도 안 된다.
- 대신 필요한 자원을 외부로부터 주입받아 사용하면 클래스의 유연성 재사용성, 테스트 용이성을 개선해준다.