# equals는 일반 규약을 지켜 재정의하라

- equals 메소드를 재정의하기는 쉽다.
- 하지만, 잘못하게 되면 끔찍한 결과를 초래하게 된다.
- 필요한 경우가 아닌 이상 재정의하지 않는 것이 최선이다.

## 재정의 X

### 1. 각 인스턴스가 본질적으로 고유하다.

- 값을 표현하는 것이 아니라 동작하는 개체를 표현하는 클래스일 경우
- 예로 Thread 클래스가 있다.

### 2. 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다.

- 즉, 동등성 비교를 할 일이 없을 경우를 말한다.

### 3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.

- 대부분의 Set, List, Map은 AbstractSet, AbstractList, AbstractMap으로부터 상속받아서 그대로 사용한다.

### 4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.

- 실수로라도 호출되는 것을 막고 싶다면 예외처리를 하면 된다.

```java
@Override
public boolean equals(Object o) {
    throw new AssertionError();
}
```

> package-private = default

## 재정의 O

### 1. 논리적 동치성을 비교해야 하는데 상위 클래스에서 equals를 재정의하지 않았을 경우

- 주로 값 클래스들이 여기에 해당한다. ex) Integer, String ∙∙∙
- Map의 키, Set의 원소 또한 정의해놔야 개발자가 생각한 대로 작동할 것이다.
- 하지만, 값 클래스라고 하더라도 Item 1에서 언급한 것과 같이 하나의 인스턴스를 재사용하게 된다면 필요하지 않다.
- 또한 Item 34에 Enum 또한 필요하지 않다.
- 이유는 인스턴스가 2개 이상 만들어지지 않기 때문에 동등성이나 동일성이나 같은 의미가 된다.

## equals 메서드를 재정의할 때 지켜야 하는 일반 규약

- 반사성(reflexivity) : null이 아닌 모든 참조 값 x에 대해, x.equals(x) == true
- 대칭성(symmetry) : null이 아닌 모든 참조 값 x, y에 대해 x.equals(y) == y.equals(x) == true
- 추이성(transitivity) : null이 아닌 모든 참조 값 x, y, 에 대해, x.equals(y) == y.equals(z) == x.equals(z) == true
- 일관성(consistency) : null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 계속 true, false면 계속 false
- null-아님 : null이 아닌 모든 참조 값 x에 대해, x.equals(null) == false

### 반사성

- 객체는 자기 자신과 같아야 한다는 것을 뜻한다.

### 대칭성

- 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.

```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String)
            return s.equalsIgnoreCase((String) o);
        return false;
    }
}
```

- cis 객체는 equals가 재정의되어 String을 동등성 처리를 구현했기 때문에 true
- 하지만 반대로 String에서 서로 같은 타입인 String만 동등성 비교를 하기에 cis 객체를 비교할 때는 false

```java
public static void main(String[] args) {
    CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
    String s = "polish";

    System.out.println(cis.equals(s)); // true
    System.out.println(s.equals(cis)); // false
}
```

- 서로 다르게 된다면 리스트에 객체를 넣고 포함하는지 검사할 때 의도한 대로 처리되지 않는다.
- false, true, exception 중 어떤 결과가 발생할지 예측할 수 없어진다.

```java
List<CaseInsensitiveString> list = new ArrayList<>();
list.add(cis);

System.out.println(list.contains(s));
```

- 이 문제를 해결하기 위해서는 CaseInsensitiveString에서 String도 같이 연동해서 동등성 비교를 하겠다는 욕심을 버려야 한다.

```java
@Override public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString &&
            ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

### 추이성

- 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같으면 첫 번째 객체와 세 번째 객체는 같아야 한다.
- 아래 예시는 상위 클래스에는 없는 새로운 필드를 하위 클래스에 추가하는 상황이다.

```java
public enum Color { RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET }

public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}

public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
}
```

- ColorPoint의 equals 메소드를 보게 되면 ColorPoint는 처리하지만 Point를 처리하지 않기 때문에 대칭성을 위반한다.

```java
Point point = new Point(1, 2);
ColorPoint colorPoint = new ColorPoint(1, 2,Color.RED);
System.out.println(point.equals(colorPoint)); // true
System.out.println(colorPoint.equals(point)); // false
```

- 그래서 둘 다 처리할 수 있도록 Point, ColorPoint 모두 처리할 수 있도록 동등성을 구현한다.

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;

    if (!(o instanceof ColorPoint))
        return o.equals(this);

    return super.equals(o) && ((ColorPoint) o).color == color;
}

System.out.println(point.equals(colorPoint)); // true
System.out.println(colorPoint.equals(point)); // true
```

- 하지만, 이 방법 또한 추이성을 위반하게 된다.
- 이 현상은 모든 객체 지향 언어의 동치관계에서 나타나는 근본적인 문제이다.
- **구체 클래스를 확장해 새로운 값을 추가하면 equals 규약을 만족시킬 방법은 존재하지 않는다.**
- 객체 지향적 추상화 이점을 포기하지 않는 한에 말이다.

```java
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
System.out.println(p1.equals(p2)); // true
System.out.println(p2.equals(p3)); // true
System.out.println(p1.equals(p3)); // false
```

- equals 메서드에 instanceof -> getClass 검사로 바꾸면 규약도 지키고 구체 클래스를 상속할 수 있다.
- 하지만, 이 방법은 리스코프 치환 법칙을 위반한다.

```java
if (o == null || getClass() != o.getClass())
    return false;
```

- 아래와 같이 Point에서 구현을 하고 CounterPoint라고 Point 클래스를 상속받은 클래스가 있다고 가정한다.
- 이때 onUnitCircle에 Point 객체가 아닌 CounterPoint 객체가 들어가면 같은 클래스가 아니라 계속 false만 반환하게 될 것이다.
- 즉 상위 타입의 객체를 하위 타입으로 객체로 치환을 했을 때 동작에 문제가 없어야 되는데 문제가 발생한 것이다.
- 예로 Timestamp, Date가 있습니다.

```java
private static final Set<Point> unitCircle = Set.of(
    new Point(1, 0), new Point(0, 1));
)

public static boolean onUnitCircle(Point p) {
    return unitCircle.contains(p);
}
```

- 우회하는 방법이 하나 있는데 **컴포지션을 이용하는 방법**이다.

```java
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint() {
        return point;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
}
```

> **컴포지션**
>
> 기존 클래스가 새로운 클래스의 구성요소가 되는 것을 말함

### 일관성

- 두 객체가 어느 하나라도 변경되지 않는다면 영원히 같아야 한다는 것이다.
- 가변 객체는 비교 시점에 따라 달라질 수 있지만, 불변 객체는 결과가 끝까지 같아야 한다.
- 클래스가 불변이든 가변이든 **equals의 판단에 신뢰할 수 없는 자원이 끼어들면 안 된다.**
    - java.net.URL의 equals는 주어진 URL과 매핑된 호스트의 IP 주소를 이용해 비교한다.
    - 호스트 이름을 IP 주소로 변환하려면 네트워크를 통해야 하는데 그 결과가 항상 같다고 보장할 수 없다.
    - 이것은 종종 문제를 일으키고 하위 호환성으로 잘못된 동작을 바로잡을 수도 없다.
    - 이런 문제를 해결하려면 항시 메모리에 존재하는 객체만을 사용하여 계산을 수행해야 한다.

### null-아님

- 모든 객체가 null과 같지 않아야 한다는 뜻이다.
- 굳이 null을 확인하지 않아도 instanceof를 이용하면 ClassCastException, null을 동시에 검증할 수 있다.

```java
// 명시적 null 검사 - 필요 없다!
@Override public boolean equals(Object o) {
	if (o == null)
		return false;
	...
}
// 묵시적 null 검사 - 이쪽이 낫다.
@Override public boolean equals(Object o) {
	if (!(o instanceof MyType))
		return false;
	MyType mt = (MyType) o;
	...
}
```

## 양질의 equals 구현 방법

- == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    - 단순한 성능 최적화용으로, 비교 작업이 복잡한 상황일 때 값어치를 한다.
- instanceof 연산자로 입력이 올바른 타입인지 확인한다.
    - 어떤 인터페이스는 자신을 구현한 (서로 다른) 클래스끼리도 비교할 수도 있다.
    - 예로 Set, List, Map, Map.Entry
- 입력을 올바른 타입으로 형변환한다.
- 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.
    - float, double을 제외한 기분 타입 필드는 == 연산자로 비교, 참조형은 equals 메서드
    - float, double은 Float.compare, Double.compare로 비교한다.
        - 이유는 Float.NaN, -0.0f, 특수한 부동 소수 값 등을 다뤄야 하기 때문이다.
    - 때로는 null도 정상값으로 취급하는 참조 타입 필드도 있을 수 있기에 그때는 Objects.equals를 사용하자
- 비용이 싼 필드를 먼저 비교한다.
    - 어떤 필드를 먼저 비교하느냐에 따라 equals 성능을 좌우하기도 한다.
    - 비용이 싼 필드가 false라면 그다음 필드는 비교할 필요가 없기 때문이다.

## equals를 다 구현했다면 세 가지만 자문해보자. 대칭적인가? 추이성이 있는가? 일관적인가?

- 자문으로 끝내지 말고 단위 테스트를 작성해 돌려보자.
- 반사성과 null-아님도 만족해야 하지만, 이 둘은 문제가 되는 경우는 별로 없다.

## 주의사항

- equals를 재정의할 땐 hashCode도 반드시 재정의하자 (아이템 11)
- 너무 복잡하게 해결하려 들지 말자.
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.
    - 구체적인 타입으로 명시한 equals는 @Override 애너테이션이 긍정 오류를 내게 하고 보안 측면에서도 잘못된 정보를 준다.

## 핵심 정리

- 꼭 필요한 경우가 아니면 equals를 재정의하지 말자.
- 재정의해야 할 때는 그 클래스의 핵심 필드를 모두 빠짐없이 다섯 가지 규약을 지키며 비교해야 한다.