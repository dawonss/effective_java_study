## 14. Comparable을 구현할지 고려하라
* Comparable 인터페이스의 유일무이한 메서드, compareTo()
* compareTo()와 Object.equals() 는 **두가지의 차이점**을 갖고 나머지는 성격이 같음
   1. compareTo()는 동치성 비교를 진행하며, 추가로 순서까지 비교 가능
   2. 제네릭하다
* **Comparable을 구현**했다 == **해당 클래스에 자연적인 순서가 존재**한다
  * 그리하여 검색, 극단 값 계산, 자동 정렬되는 컬렉션 관리도 쉽게 가능해진다
  * 자바 플랫폼 라이브러리의 모든 **값클래스와 열거타입이 Comparable 인터페이스를 구현**하고 있다
  * 알파벳, 숫자, 연대와 같이 **순서가 명확한 값 클래스를 작성할 땐 반드시 구현하도록 한다**
### 1️⃣ compareTo의 일반 규약
* 이 객체와 **주어진 개체의 순서를 비교**. 만약 주어진 객체가 더 **크면 음의 정수**, **같다면 0**, 더 **작다면 양의 정수**를 반환
* 만약 비교할 수 없는 타입의 객체가 주어진 경우, **ClassCastException 반환**
1.  Comparable을 구현한 클래스는 모든 x,y 에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x)).
    * 만약 x.compareTo(y)가 예외를 던지면 y.compareTo(x)도 마찬가지
    * 즉, **두 객체의 참조 순서를 바꿔도 예상한 결과가 나와야 한다**
2. x.compareTo(y) > 0 && y.compareTo(z) > 0 이라면 x.compareTo(z) > 0 이다
    * Comparable을 구현한 클래스는 **추이성을 보장**해야 한다.
3.  Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z)) 
    * 즉, **크기가 같은 객체들끼리는 어떤 객체와 비교해도 항상 같아야 한다**
4. (x.compareTo(y) == 0) == (x.equals(y) == 0) 이 규약은 필수는 아니지만 지키는 게 좋음
    * 즉 **compareTo로 수행한 동치성 검사 결과가 equals와 같아야 한다**
    * 다시말해 compareTo()로 줄지은 순서와 equals의 결과가 일관되게 되어야 한다
    * 왜냐면 **정렬된 컬렉션에 넣으면** 해당 **컬렉션이 구현한 인터페이스(Collection, Set, Map)에 정의된 동작과 엇갈리게 동작**하기 때문
        * 이 인터페이스들은 **동치성을 비교**할 때 **equals() 대신 compareTo()를 사용**하기 때문
* 따라서 **compareTo 메서드로 수행하는 동치성 검사**도 equals 규약과 똑같이 **반사성, 대칭성, 추이성을 충족**해야 한다. 그래서 **주의 사항과 우회법도 같다**
### 2️⃣ compareTo의 작성 요령
* equals와 비슷하지만 몇개만 주의
1. 인수 타입이 컴파일 시 정해지기 때문에 입력인수의 타입을 확인하거나 형변환을 해줄 필요가 없음
2. null을 인수로 하여 호출하면 NullPointerException을 던져야 함
### 3️⃣ Comparator, 비교자
* **compareTo()는 각 필드의 동치여부가 아닌 순서를 비교**
* 객체 참조 필드를 비교하려는 경우에는 객체 내부의 필드도 검사하기 위해 compareTo메서드를 재귀적으로 호출한다
* **Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 Comparator을 사용**하자.
    * 이 때 Comparator는 직접 만들수도 있고 자바가 제공하는 걸 사용해도 된다
```java
// 자바가 제공하는 비교자 사용
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
  public int compareTo(CaseInsecsitiveString cis) {
    return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
  }
  ...
}
```
> "CaseInsensitiveString implements Comparable<CaseInsensitiveString>"는 CaseInsensitiveString의 참조는 CaseInsensitiveString 참조와만 비교할 수 있다는 뜻으로 구현할 때는 같은 타입의 참조만 비교가 가능함을 명시하자
### 4️⃣ 주의사항
1. compareTo()에서 관계 연산자 **>, <는 사용하지 않는다.** 박싱된 기본 타입 클래스의 **정적 메서드인 compare을 사용**하자
2. **클래스에 핵심 필드가 많다면, 가장 핵심적인 필드부터 비교**하라. 그리고 그 비교 결과가 0이 아니라면 순서가 정해진 것이므로 그 결과를 반환하자
```java
// PhoneNumber의 compareTo 1 : 기본 타입 필드가 여럿일 때의 비교자
public int compareTo(PhoneNumber p){
  int result = Short.compare(areaCode, p.areaCode); //첫번째로 중요
  if (result == 0) {
    result = Short.compare(prefix, p.prefix); //두번째로 중요
    if (result == 0)
      result = Short.compare(lineNum, p.lineNum) //세번째로 중요
  }
  return result;
}
```
> 정적 메서드 Short.compare() 이용하여 비교
### 5️⃣ 비교자 생성 메서드
* 자바 8부터 **Comparator 인터페이스가 비교자 생성 메서드를 이용**(사용)하여 **메서드 연쇄 방식으로 비교자를 생성**할 수 있도록 하였다
    * 하지만 약간의 성능 저하가 따를 수 있다
```java
// PhoneNumber의 compareTo 2 : 비교자 생성 메서드를 활용한 비교자
private static final Comparator<PhoneNumber> COMPARATOR = comparingInt((PhoneNumber p) -> p.areaCode)
    .thenComparingInt(p -> p.prefix)
    .thenComparingInt(p -> p.lineNum);

public int compareTo(PhoneNumber p) {
  return COMPARATOR.compare(this, p)
}
```
> 클래스 초기화 시 **COMPARATOR 라는 비교자를 생성**하게 됨 <br>
> 그 과정에 **비교자 생성 메서드 2개 이용** <br>
> 1. **comparingInt()** : 객체 참조를 int 타입 키에 매핑하는 키 추출함수(람다 이용)를 인수로 받아 받은 키를 기준으로 순서를 비교 <br>
> 2. **thenComparingInt()** : 첫번째 비교자를 적용한 다음 새 키를 기준으로 추가 비교. 연달아 호출 무한대로 가능 (코드에선 2번 사용) <br>
1. 숫자용 기본 타입 비교자 생성 메서드
    * comparingInt와 thenComparingInt를 사용
    * long, double이라면 comparingInt와 thenComparingInt 변형인 comparingLong, comparingDouble... 등을 사용
2. 객체 참조용 비교자 생성 메서드
    * comparing(총 2개. 1. 키 추출자로 그 키의 자연적 순서를 사용하여 비교, 2. 키추출자와 비교할 비교자를 인수로 받음)
    * thenComparing(총 3개. 1. 비교자로 부차순서 정하기, 2. 키 추출자로 자연적 순서에 맞는 보조 순서 정하기, 3. 키추출자와 비교할 비교자 인수로 받음)
```java
// 잘못된 코드 : 해시코드 값의 차를 기준으로 비교 - 추이성을 위반
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int compare(Object o1, Object o2) {
    return o1.hashCode() - o2.hashCode();
  }
};
```
> **정수 오버플로우**를 일으키거나 **부동소수점 계산 방식에 따른 오류**가 날 수 있음
```java
// 올바른 예시1 : 정적 compare 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  public int compare(Object o1, Object o2) {
    return Integer.compare(o1.hashCode(), o2.hashCode());
  }
};
```
> 정적 compare 메서드인 **Integer.compare()을 이용**하여 해시 코드를 비교
```java
// 올바른 예시2 : 비교자 생성 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```
> **객체의 해시코드 메서드를 기준으로 결과를 int 타입 키에 매핑**하여 비교
### ❗ 핵심 요약
1. **순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하자**
2. compareTo 메서드에서 필드의 값을 비교할 때는 <, > 연산자를 쓰지 않도록 한다
3. 박싱된 기본 타입 클래스가 제공하는 **정적 compare 메서드** or Comparator 인터페이스가 제공하는 **비교자 생성 메서드를 사용**하자
