## 13. clone 재정의는 주의해서 진행하라
Cloneable : 복제해도 되는 클래스 임을 명시하는 용도의 인터페이스
* clone 메서드는 Object에 선언되어 있고 이는 protected이다
* Cloneable을 구현하는 것만으로는 외부 객체에서 clone메서드를 호출할 수 없음
### 1️⃣ Cloneable 인터페이스가 하는 일
: Object.clone() 의 동작 방식을 결정
* Cloneable을 구현한 클래스의 인스턴스에서 clone 호출 시 그 객체의 필드 하나하나 다 복사한 객체를 반환하고 그렇지 않으면 CloneNotSupportedException을 발생시킴
* 실무에서 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공하며, 사용자는 당연히 복제가 제대로 이뤄지리라 기대함
    * 하지만 이는 생성자를 호출하지 않고도 객체를 생성할 수 있게 되어 위험하고 모순적인 메커니즘이 탄생하게 되는 것이다
### 2️⃣ clone 메서드의 일반 규약 (Object 명세에 나타난)
1. **이 객체의 복사본을 생성해 반환**한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다.
    * x.clone() != x ➡️ true
    * x.clone().getClass == x.getClass() ➡️ true
    * x.clone().equals(x) ➡️ ture
2. 관례상 이 메서드가 **반환하는 객체는 super.clone을 호출**해 얻어야 한다.
    * x.clone().getClass() == x.getClass()
3. 관례상 **반환된 객체와 원본 객체는 독립적**이어야 한다. 이를 만족하기 위해 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.
* 강제성이 없다는 점을 제외 하면 **생성자 연쇄와 비슷한 메커니즘**
    * 즉 clone메서드가 super.clone이 아닌 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러는 아무런 불평이 없을 것
* 하지만 **생성자를 호출할 경우** 이 클래스의 하위 클래스에서 super.clone을 호출한다면 **상위 클래스 타입의 객체를 반환**하여 하위 클래스의 clone 메서드가 제대로 동작하지 않게 된다.
### 3️⃣ clone 메서드 구현
1. 제대로 동작하는 **clone 메서드**
    1. 먼저 **super.clone 호출** (원본 필드와 똑같은 값을 갖는 클래스 내의 모든 필드들)
    2. super.clone에서 **얻은 객체를 하위 클래스 타입으로 형변환**
```java
// 가변 상태를 참조하지 않는 클래스용 clone메서드
@Override public PhoneNumber clone() {
   try{
      return (PhoneNumber) super.clone();
   } catch (CloneNotSupportedException e) {
      throw new AssertionError(); //일어날 수 없는 일
   }
}
```
> 재정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있다
2. **가변 객체를 참조하는 클래스**의 경우
    * Stack에는 원소를 담을 배열인 **Object[] element 필드**가 존재
    * 이를 단순히 **super.clone 만 진행**한다면 element 필드는 **원본 Stack 인스턴스와 똑같은 배열을 참조**할 것
        * 따라서 프로그램이 이상하게 동작하거나 NullPointerException이 발생
    * clone 메서드는 사실상 생성자와 같은 효과를 낸다. 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변시을 보장해야하는 것이다
        * 그러므로 Stack의 **clone 메서드가 제대로 동작하려면 스택 내부 정보를 복사**하여야 한다.
    * final 필드의 경우 새로운 값을 할당할 수 없기 때문에 복제할 수 있는 클래스를 만들기 위해서는 일부 필드에서 final 한정자를 제거해야 할 수 있다.
```java
// 가변 객체를 참조하는 클래스 clone 메서드
@Override public Stack clone() {
   try {
      Stack result = (Stack) super.clone();
      result.elements = elements.clone();
      return result;
   } catch (CloneNotSupportedException e){
      throw new AssertionError();
   }
}
```
> **배열의 clone은** 런타임 타입과 컴파일 타입 모두 **원본 배열과 똑같은 배열을 반환하므로 형변환은 필요하지 않다** <br>
> 그러므로 배열을 복제할 때는 배열의 clone 메서드를 사용하는 것이 좋다
3. **복잡한 가변 상태를 갖는 클래스**의 경우
    * HashTable을 보면 Entry[]의 buckets 필드가 존재
    * **Entry 클래스는 다음 entry 정보를 필드로 저장**하고 있음 꼬꼬무로 진행
```java
// 잘못된 clone 메서드 - 가변 상태를 공유함
@Override public HashTable clone() {
   try {
      HashTable result = (HashTable) super.clone();
      result.buckets = buckets.clone();
      return result;
   } catch (CloneNotSupportedException e){
      throw new AssertionError();
   }
}
```
> 배열이라고 **배열의 clone 메서드를 사용**하면 **원본과 같은 연결 리스트를 복제본이 참조**하게 되어 문제가 발생할 수 있다. 
```java
// 개선한 방법 - 복잡한 가변 상태를 갖는 재귀적 clone 메서드
public class HashTable implements Cloneable {
   private Entry[] buckets = ...;
   ...
   private static class Entry {
      ...
      Entry deepCopy() {
         return new Entry(key, value, next == null? null : next.deepCopy());
      }
   }
   @Override public HashTable clone() {
      try {
         HashTable result = (HashTable) super.clone;
         result.buckets = new Entry[buckets.length];
         for (int i = 0; i < buckets.length; i++)
            if(buckets[i] != null)
               result.buckets[i] = buckets[i].deepCopy();
         return result;
      } catch (CloneNotSupportedException e){
      throw new AssertionError();
      }
   }
   ...
}
```
>  HashTable.Entry 클래스는 **깊은 복사를 지원**하도록 추가 <br>
> clone 메서드는 새로운 버킷 배열을 할당하고 원본 버킷 배열을 순회하면서 **비어있지 않은 버킷의 깊은 복사를 재귀적으로 호출**하여 채워 넣는다 <br>
> 간단한 방법이지만 리스트의 원소 수만큼 스택 프레임을 소비하여 **리스트가 길면 스택 오버플로우**를 일으킬 수 있다.
```java
// 더 개선한 방법 - 반복자를 이용한 clone 메서드
Entry deepCopy() {
   Entry result = new Entry(key, value, next);
   for (Entry p = result; p.next != null; p = p.next)
      p.next = new Entry(p.next.key, p.next.value, p.next.next);
   return result;
}
```
4. **super.clone 호출 후 원본 객체 상태 다시 생성**하는 방법
    1. super.clone을 호출하여 얻은 객체의 모든 필드를 초기 상태로 저장
    2. 그 후 원본 객체의 상태를 다시 생성하는 고수준 메서드들을 호출
    * 간단하지만 처리 속도가 비교적 느리고 **필드 단위 객체 복사를 우회**하여 전체 Cloneable 아키텍처와는 어울리지 않은 방식이다
### 4️⃣ 주의 사항
* 생성자에서와 마찬가지로 clone 메서드에서도 재정의 될 수 있는 메서드를 호출하면 안된다
* public인 clone 메서드에서는 throws 절을 없애야 한다
* 상속을 위해 사용되는 클래스 설계 방식 두 가지 모두 상속용 클래스는 Cloneable을 구현하면 안된다
### 5️⃣ 정리
1. **Cloneable을 구현하는 모든 클래스는 clone을 재정의** 해야 한다
2. 이때 **접근 제한자는 public**, **반환 타입은 클래스 자신**으로 변경한다
3. 가장 **먼저 super.clone을 호출**한 후 **필요한 필드를 적절히 수정**한다
    * 객체 내부 '깊은 구조'에 숨어 있는 모든 가변 객체를 복사하고, 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 하자
4. 기본 타입 필드와 분변 객체 참조만 갖는 클래스의 경우 아무 필드도 수정할 필요가 없다
### 6️⃣ 복사 생성자와 복사 팩터리
* 복사 생성자? 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자
* **Cloneable을 구현하여 사용하지 않아도 복사생성자와 복사 팩터리를 이용해 더 나은 방식으로 객체 복사를 할 수 있다**
    * 이 방식은 언어 모순적이지 않으며 생성자를 사용하여 객체 생성 메커니즘이 안전하며 형변환도 필요하지 않는 등 더 나은 면이 많다
* 또한 해당 클래스가 구현한 '인터페이스'타입의 인스턴스를 인수로 받을 수 있다
    * 인터페이스 기반 복사 생성자와 복사 팩터리인 **변환 생성자와 변환 패터리를 이용하면 클라이언트는 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다**
### ❗ 핵심 요약
1. 새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안되며, 새로운 클래스도 이를 구현해서는 안된다.
2. **복제 기능은 생성자와 팩터리를 이용**하는 게 최고이지만 **배열만은 clone 메서드 방식**이 가장 깔끔하고 합당한 예외이다
