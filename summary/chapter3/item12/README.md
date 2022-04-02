# toString을 항상 재정의하라

- Object의 기본 toString은 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없다.
- 단순히 `클래스 이름@16진수로 표시한 해시코드`를 반환한다.
- toString 일반 규약에 따르면 **간결하면서 사람이 읽기 쉬운 형태의 유익한 정보**를 반환해야 한다.

## toString을 잘 구현한 클래스는 사용하기에 훨씬 즐겁고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다.

- toString 메서드는 해당 상황에 자동으로 불린다.
    - println, printf
    - 문자열 연결 연산자(+)
    - assert 구문에 넘길 때
    - 디버거가 객체를 출력할 때
- 위 말은 즉, 사용자가 직접 호출하지 않아도 자동으로 어디선가 쓰일 거라는 이야기이다.
- 재정의하지 않으면 쓸모없는 메시지가 출력되거나 쌓일 것이다.

## 실전에서 toString은 그 객체가 가진 주요 정보 모두를 반환하는 게 좋다.

- 객체가 거대하거나 객체의 상태가 문자열로 표현하기에 적합하지 않다면 쉽지 않기에 요약 정보를 담아 표현하는 것이 좋다.
    - "{Jenny=707-867-5309}" -> "맨해튼 거주자 전화번호부(총 1487536개)"
- 이상적으로 스스로를 완벽히 설명하는 문자열이어야 한다.

## toString을 구현할 때면 반환 값의 포맷을 문서화할지 정해야 한다.

- 전화번호나 행렬 같은 값 클래스는 문서화하는 것을 권한다.
- 포맷을 명시하면 그 객체는 표준적이고, 명확하고, 사람이 읽을 수 있게 된다.
- 따라서 그 값을 입출력으로 사용하거나 CSV 파일처럼 사람이 읽을 수 있는 데이터 객체로 저장할 수 있다.
- 단점은 포맷을 한번 명시하면 (그 클래스가 많이 쓰인다면) 평생 그 포맷에 얽매이게 된다.

## 포맷을 명시하든 아니든 여러분의 의도는 명확히 밝혀야 한다.

- 포맷을 명시하려면 아주 정확하게 해야 한다.

```java
/**
* Returns the string representation of this phone number.
* The string consists of twelve characters whose format is
* "XXX-YYY-ZZZZ", where XXX is the area code, YYY is the
* prefix, and ZZZZ is the line number. Each of the capital
* letters represents a single decimal digit.
*
* If any of the three parts of this phone number is too small
* to fill up its field, the field is padded with leading zeros.
* For example, if the value of the line number is 123, the last
* four characters of the string representation will be "0123".
*/
@Override public String toString() {
    return String.format("%03d-%03d-%04d",
            areaCode, prefix, lineNum);
}
```

- 포맷을 명시하지 않기로 했다면 다음처럼 작성해야 한다.
- 이러한 설명을 보고도 이 포맷에 맞춰 코딩해서 피해를 본다면 자신을 탓할 수밖에 없다.

```java
/**
* 이 약물에 관한 대략적인 설명을 반환한다.
* 다음은 이 설명의 일반적인 형태이나,
* 상세 형식은 정해지지 않았으며 향후 변경될 수 있다.
*
* "[약물 #9: 유형=사랑, 냄새=테레빈유, 겉모습=먹물]"
*/
@Override public String toString() { ... }
```

## 포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자

- getter와 같은 멤버 변수에 접근할 수 있는 인터페이스를 제공해야 한다.
- 그렇지 않으면 toString의 반환값으로 파싱하여 사용해야 하기 때문 상당히 비효율적이다.

```java
public class PhoneNumber {
    private int areaCode;
    private int prefix;
    private int lineNum;

    public int getAreaCode() { return areaCode; } // API 제공
    public int getPrefix() { return prefix; } // API 제공
    public int getLineNum() { return lineNum; } // API 제공

    @Override public String toString() {
        return String.format("%03d-%03d-%04d",
                areaCode, prefix, lineNum);
    }
}
```

## toString 재정의 필요 시점

- 필요 없는 경우
    - 정적 유틸리티 클래스 (아이템 4)
    - 열거 타입 (아이템 34ㄴ)
- 필요한 경우
    - 하위 클래스들이 공유해야 할 문자열 표현이 있는 추상 클래스 (ex. 대다수의 컬렉션 구현체)

## 핵심 정리

- 모든 구체 클래스에서 Object의 toString을 재정의하자.
- 하지만, 상위 클래스에서 이미 알맞게 재정의한 경우는 예외다.
- toString을 재정의한 클래스는 사용하기 좋고 시스템 디버깅을 하기 쉽게 해준다.
- 재정의할 때 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.