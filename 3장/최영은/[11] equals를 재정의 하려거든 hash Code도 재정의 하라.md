## 11. equals를 재정의 하려거든 hashCode도 재정의 하라
* equals를 재정의한 클래스 모두에서 hashCode도 재정의 해야 한다
* hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMay or HashSet 같은 컬렉션의 원소로 사용하는 경우 문제를 일으킨다
### 1️⃣ Object 명세 내 이와 관련된 규약
 1. equals 비교에 사용되는 정보가 변경되지 않으면 hashCode메서드는 일관되게 같은 값을 반환해야한다
 2. **equals(Object)가 true**면 두 객체의 **hashCode는 같은 값을 반환**해야한다.
    * 즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.
 3. equals(Object)가 false여도 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다
    * but, **다른 객체에 대해서는 다른 hashCode를 반환하는 것이 해시 테이블 성능에 좋다** (그래야 해시 테이블 성능이 좋아진다)
### 2️⃣ equals는 재정의 , hashCode는 재정의 하지 않았을 때 발생하는 일
: equals는 true라고 해도, Object의 기본 hashCode()에서는 이들이 전혀 다르다고 판단하여 서로 다른 값을 반환 한다. (2번 규약) <br>
```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(070, 861, 5309), "제니");
m.get(new PhoneNumber(707, 867, 5309));
```
> 서로 논리적으로 동치이지만 서로 다른 두 개의 PhoneNumber 인스턴스가 사용되기 때문에 서로 다른 hashCode를 반환하게되어 결과적으로 null을 반환함
### 3️⃣ 안좋은 hashCode 구현의 예
: 그렇다면 모든 객체가 같은 값을 반환하게 하는 것은 어떨까? 논리적으로 동치인 객체도 hashCode가 같은 값이 반환되므로 2번 규약을 만족할 수 있다 <br>
```java
// 최악의 hashCode 구현 - 사용 금지 절대 금지
@Override public int hashCode() { return 42; }
```
> 하지만 항상 똑같은 hashCode값을 반환하여 모든 객체가 해시 테이블의 버킷 하나에 담겨 연결리스트 처럼 작동하여 평균 수행 시간이 O(1)에서 O(n)으로 느려짐 <br>
> ➡️ 3번 규약을 만족할 수 없게 됨
### 4️⃣ 좋은 hashCode를 작성하는 요령
1. **int 변수 result를 선언하고 값 c로 초기화**
2. 해당 **객체**의 나머지 **핵심 필드 f 각각에 대해 작업을 수행**
   1. **해당 필드의 해시 코드 c를 계산**
      * 기본 타입 필드인 경우, _Type_.hashCode(f) 수행
      * 참조 타입 필드인 경우, 해당 필드의 hashCode를 재귀적으로 호출
        * 계산이 복잡해지는 경우 필드의 표준형을 만들어 그 표준형의 hashCode호출
        * 필드 값이 null이라면 0 사용
      * 배열인 필드의 경우, 핵심 원소 각각을 별도 필드처럼 취급. 각각 재귀적으로 호출하여 2.2처럼 갱신 
   2. 2.1에서 계산한 **해시코드 c로 result를 계산**
      * ex) result = 31*result + c;
3. **result 반환**
    * 이후 검증을 위한 단위 테스트를 작성하여 논리적 동치인 인스턴스가 서로 다른 hashCode를 반환하지 않는지 확인 (2번 규약 만족하는지 확인)
    * **파생 필드**와 **equals() 비교에 사용되지 않는 핵심 필드가 아닌 필드**는 **반드시 제외**해야 한다. 그렇지 않으면 2번 규약을 어기게 될 수 있다
    * 2.1이 31* 인 이유? 비슷한 필드가 여러개여도 해시 효과가 커질 수 있기 때문
        * ex) String.hashCode에 곱셈이 없는 경우, 모든 아나그램의 해시코드는 동일하다
        * 아나그램 : 구성하는 철자가 같고 순서만 다른 문자열
```java
// 전형적인 hashCode 메서드
@Override public int hashCode() {
  int result = Short.hashCode(areaCode);
  result = 31 * result + Short.hashCode(prefix);
  result = 31 * result + Short.hashCode(lineNum);
  return result;
}
```
> 비결정적 요소가 없으므로 2번 규약 만족 <br>
> 단순하고 빠르며 해시 테이블 값 분배도 적절히 될 수 있음 <br>
> 만일 해시 충돌이 더 적고자 하면 구아바의 com.google.common.hash.Hasing을 참고한다
### 5️⃣ Object.hash
: Object는 임의의 개수 만큼 객체를 받아 해시 코드를 계산해주는 정적 메서드 hash를 제공하고 있음
```java
@Override public int hashCode() {
  return Objects.hash(lineNum, prefix, areaCode);
}
```
> 이를 이용하면 한 줄로 위의 hashCode() 기능과 비슷한 수준으로 작성할 수 있음 <br>
> but, 느림. 성능에 민감하지 않은 경우에만 사용하도록 하라 <br>
### 6️⃣ 해시 코드 캐싱
* 클래스가 불변이고 해시코드 계산 비용이 큰 경우 매번 새로 계산하기보다 캐싱 방식을 고려해보도록 한다
* 만일 이 타입의 객체가 해시의 키로 사용될 것 같다면, 인스턴스 생성 시 해시코드를 계산
* 그렇지 않다면, hashCode가 불릴 때 계산되도록 지연 초기화를 사용할 수도 있음
  * 이 경우에는 스레드 안전성에 신경을 써야한다.
### 7️⃣ 그 외 팁
1. 성능을 높인다고 해시코드를 계산할 때 핵심 필드를 생략하지 마라
    * 속도는 빨라질 수 있지만 해시 품질이 나빠져 해시 테이블 성능이 심각히 저하될 수 있다
2. hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 마라
    * 클라이언트가 이 값에 의지하게 될 수 있다
    * 추후 계산 방식을 바꾸기가 어렵다
    * 알려지지 않으면 추후 해시 기능에서 결함을 발견하거나 더 나은 해시 방식을 알아낸 경우, 다음 릴리즈에서 수정 할 수 있음
### ❗핵심 요약
1. **equals를 재정의 할 때는 반드시 hashCode도 재정의** 하여 프로그램이 제대로 동작 되도록 하고, **재정의 시 Object의 API 문서 내 일반 규약을 따라야 한다**. 
2. 필요 시, AutoVaule 프레임 워크의 equals와 hashCode를 자동으로 생성해주는 기능을 이용하라
