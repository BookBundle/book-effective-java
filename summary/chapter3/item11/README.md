# equals를 재정의하려거든 hashCode도 재정의하라

> Object 명세
>
> - equals 비교에 사용되는 정보가 변경되지 않았다면
>     - 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관된 값을 반환해야 한다.
>     - 단, 애플리케이션이 다시 실행된다면 값이 달라져도 상관없다.
> - **equals(Object)가 두 객체를 같다고 판단했다면**
>     - **두 객체의 hashCode는 똑같은 값을 반환해야 한다.**
> - equals(Object)가 두 객체를 다르다고 판단했더라도
>     - 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다.
>     - 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

- hashCode를 재정의하지 않았을 때 크게 문제가 되는 조항은 두 번째이다.
- PhoneNumber가 hashCode를 재정의하지 않는다면 두 객체는 서로 다른 해시코드를 반환한다.
- 그래서 엉뚱한 해시 버킷에 가서 객체를 찾으려 하기 때문에 값을 찾을 수 없다.

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");
m.get(new PhoneNumber(707, 867, 5309)); // null
```

## 해시함수 구현 안좋은 예

- 모든 객체가 같은 해시코드를 반환하기 때문에 하나의 버킷에 모든 데이터가 들어가게 된다.
- 이건 해시테이블 자료구조 이점을 전혀 사용하지 못하는 것으로 값을 찾으려고 할 때 O(n)이라는 비효율적인 속도를 가진다.

```java
@Override public int hashCode() { return 42; }
```

## 좋은 해시 작성 방법

- 서로 다른 인스턴스에 다른 해시코드를 반환한다.
- 이상적인 해시 함수는 주어진 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다.

### 좋은 hashCode 작성하는 간단한 요령

1. int 변수 result를 선언한 후 값 c로 초기화한다. 이때 c는 해당 객체의 첫 번째 핵심 필드를 단계 2.a 방식으로 계산한 해시코드이다.
1. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
    - a. 해당 필드의 해시코드 c를 계산한다.
        - i. 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스이다.
        - ii. 참조 타입 필드면서 이 클래스의 equals 메소드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다.
        - iii. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.b 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0을 추천한다)를 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.
    - b. 단계 2.a에서 계산한 해시코드 c로 result를 갱신한다. 코드로는 다음과 같다.
    - b. result = 31 * result + c;
        - 곱할 숫자가 31인 이유는 31이 홀수이며 소수이기 때문이다. 만약 짝수이고 오버플로우가 발생한다면 정보를 잃는다.
1. result를 반환한다.

## 전형적인 hashCode 메서드

- 이 메서드는 핵심 필드 3개만을 사용해 간단한 계산만 수행한다.
- 이 과정에 비결정적인 요소가 없기 때문에 동치인 동일 인스턴스들은 같은 해시코드를 가지게 된다.

```java
@Override public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashcode(lineNum);
    return result;
}
```

## 한 줄짜리 hashCode 메서드 - 성능이 살짝 아쉽다.

- 해시 충돌이 더 적은 방법을 사용하겠다면 com.google.common.hash.Hashing을 참고해라.
- 또는 Objects 클래스의 hash 함수를 이용해 흉내 낼 수 있다. 하지만 속도는 더 느리기에 성능에 민감하지 않을 때만 사용하는 것이 좋다.
    - 입력 인수를 담기 위한 배열이 만들어지고, 입력 중 기본 타입이 있다면 박싱과 언박싱도 거쳐야 하므로 느리다.

```java
@Override public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```

## 해시코드를 지연 초기화하는 hashCode 메서드 - 스레드 안정성까지 고려해야 한다.

- 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하는 것이 아닌 캐싱을 이용해야 한다.
- 해당 클래스의 객체가 주로 해시의 키로 사용될 것 같다면 인스턴스가 만들어질 때 해스코드를 계산해둬야 한다.
- 해시의 키로 사용되지 않은 경우라면 지연 초기화 전략을 사용한다.

```java
private int hashCode; // 자동으로 0으로 초기화된다.

@Override public int hashCode() {
	int result = hashCode;
	if(result == 0){
		result = Short.hashCode(areaCode);
		result = 31 * result + Short.hashCode(prefix);
		result = 31 * result + Short.hashCode(lineNum);
		hashCode = result;
	}
	return result;
}
```

## 주의해야 할 점

- 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다.
    - 속도는 빨라지겠지만, 해시 품질이 나빠져 해시테이블의 성능을 심각하게 떨어뜨릴 수 있다.
    - 특히 어떤 필드는 특정 영역에 몰리는 인스턴스들을 해시코드를 넓은 범위로 고르게 퍼트려주는 효과가 있을지도 모른다.
    - 이런 필드를 생략했다면 많은 인스턴스가 하나의 해시코드에 집중되어 해시테이블 속도가 선형으로 느려질 것이다.
- hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 다음에 계산 방식을 바꿀 수도 있다.

## 핵심 정리

- equals를 재정의할 때는 hashCode도 반드시 재정의해야 한다.
- 재정의한 hashCode는 Object의 API 문서에 기술된 일반 규약을 따라야 한다.
- 서로 다른 인스턴스라면 해시코드도 서로 다르게 구현해야 한다.
- item 10에서 언급한 AutoValue 프레임워크를 사용하면 멋진 equals와 hashCode를 자동으로 만들어준다.