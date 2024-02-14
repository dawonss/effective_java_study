# 아이템11- equals를 재정의 하려거든 hashCode도 재정의하라

## HashCode 규약

- equals 비교에 사용하는 정보가 변경되지 않았다면 hashCode는 매번 같은 값을 리턴해야한다. (정보가 변경되거나, 애플리케이션을 다시 실행했다면 달라질 수 있다. )
- **두 객체에 대한 equals가 같다면, hashCode의 값도 같아야 한다.**
- 두 객체에 대한 equals가 다르더라도, hashCode의 값은 같을 수 있지만 해시테이블 **성능을 고려해 다른 값을 리턴하는 것이 좋다.**

→ hashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 두번째 이다

**두 객체에 대한 equals가 같다면, hashCode의 값도 같아야 한다.** 

아이템에서 보았듯이 equals는 물리적으로 다른 두 객체를 논리적으로 같다고 할 수 있다. 

## HashCode는  무엇인가?

**Object클래스의 hashCode() 메서드**

Java에서 사용되는 **hashcode는 객체를 식별하기 위한 ID**이다. Java의 모든 객체는 JVM에 의해 고유번호가 생성되고 이를 hashcode라고 한다. 

해시코드는 32비트 고유한 정수값으로 객체와 다른 객체를 구별하기 위해 사용되며, 객체의 내부 주소를 정수로 변환된 값이다. 

### **해시코드를  사용하는 이유는 ?**

Java에서 해시코드를 기반으로 데이터를 관리하는 이유는 데이터 검색, 추가, 제거하는 작업이 쉬워지기 때문이다. 시간복잡도가 O(1)이므로 빠르게 동작한다.

Hashcode를 사용하는 이유 중에 하나는, 객체를 비교할 때 드는 비용을 낮추기 위해서다. JAVA에서 2개의 객체가 같은지 비교할 때 `equals()`를 사용하는데, 여러 객체를 비교할 때 `equals()`를 사용하면 Integer를 비교하는 것에 비해 많은 시간이 소요된다.

**올바른 hashcode() 작성법**

올바른 hashcode 메서드는 서로 다른 인스턴스에 다른 해시코드를 반환하도록 작성해야 한다. 

**해시 코드 생성 원리**

각 `PhoneNumber`  인스턴스의 주소 값은 당연히 다르기 때문에 논리적 동치인 두 객체가 서로 다른 해시 코드를 반환한다. 

설사 두 인스턴스를 같은 해시 버켓에 담았더라도, `HashMap`은 해시코드가 다른 엔트리끼리는 동치성 비교를 하지 않도록 최적화 되어 있어서 

`get()` 메서드는 여전히 null을 반환한다. 

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");
m.get(new PhoneNumber(707, 867, 5309));   //제니가 아닌 null 반환
```

### **컬렉션에서의 두 객체 동등성 판별**

컬렉션 프레임워크에서의 `HashSet`, `HashMap`, `HashTable` 은 다음과 같은 방법으로 두 객체의 동등성을 판별한다. 

1. 우선적으로 해시 코드를 비교한다. 
2. 그 다음 같은 해시 코드 객체에 대해 `equals` 를 통해 동등성까지 비교한다. 

따라서 애초에 해시 코드가 다르다면 Map 에서 같은 키로 간주하지 않기 떄문에 기존 데이터에 덮어씌워지는 것이 아닌, 새로운 객체가 생성되어 객체가 2개 저장 되버리는 문제가 발생한다. 

**1) 절대 사용해서는 안되는 hashcode 구현**

```java
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class home{
    private int address;
    private String city;
    private String street;    

    @Override
    public int hashCode(){
      return 1;
  }

    @Override
    public boolean equals(Object o){
        if(this == null) return true;
        if(o == null || this.getClass() != o.getClass()) return false;
        HashCodeTest hash = (HashCodeTest) o;
        return address == hash.address
                && (hash.equals(hash.city) && hash.equals(hash.street));
    }

}
```

home 클래스는 규약에 따라서 equals()와 hashcode()를 재정의 하였다. 

하지만 **논리적으로 같다고 정의되는 모든 객체들에서 똑같은 해시코드를 반환한다.** 

이는 모든 객체가 해시테이블 버킷 하나에 담겨 마치 연결 리스트(linked list)처럼 동작하기 때문에 

평균수행시간이 O(1)인 해시테이블이 O(n)으로 느려져서 hashTable의 성능을 퇴화시킨다. 

또한 버킷에 대한 overflow가 발생하여 데이터가 누락될 수도 있다. 

**2) 한줄짜리 hashcode 메서드(성능이 아쉽다)**

```java
@Override
    public int hashCode(){
      return (int)address * city.hashCode() * street.hashCode();
  }
```

해시코드를 개선하기 위해서 home 클래스에 존재하는 모든 필드를 곱하여 해시코드를 계산하기 때문에 hashcode의 중복 확률이 낮아진다. 

하지만 속도는 더 느리다. 입력인수를 담기 위해서 배열이 만들어지고, 입력중 기본타입이 있다면 박싱과 언박싱도 거쳐야 하기 때문이다. 

3**) 전형적인 hashcode 메서드**

```java
@Override
    public int hashCode(){
			int hash = 17;
			hash = 31 * hash +(int) address;
			hash = 31 * hash +(city == null ? 0 : city.hashCode());
			hash = 31 * hash +(street== null ? 0 : street.hashCode());
      return hash;
  }
```

해시코드를 위와 같이 사용 하는 것이 가장 보편적이다.

**숫자 31을 곱해주는 이유는 무엇일까?** 

- 31이 홀수이면서 소수(prime)이기 때문이다. 만약 이 숫자에 짝수를 곱하게 되면 시프트 연산과 같은 효과를 내어 오버플로우가 발생하여 정보를 잃게 된다.  31을 이용하면, 이 곱셈을 비트 시프트 연산과 뺄셈으로 대체해 최적화 할 수 있다.
- 곱셈을 비트 시프트 연산과 뺄셈으로 대체할 수 있으며 VM에서 자동으로 최적화 할 수 있다.

**`31 * x`**는 **`(x << 5) - x`**로 최적화될 수 있다.

### 시프트 연산

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a46c8613-8c96-4455-8720-6f5234cbb266/ba8d81ad-29f1-411f-b624-d79d38a274e4/Untitled.png)

4**) intelliJ에서 만들어 hashcode 메서드**

```java
@Override
    public int hashCode() {
        return Objects.hash(address, city, street);
    }
```

IDE에서 커스텀한 hashcode를 구현할 시에는 다음과 같다.

Object.hash() 함수를 이용하면 간편하지만 내부적으로 **AutoBoxing**이 일어나서 속도는 더 느리다. 

- 오토 박싱(Auto Boxing)

자바 컴파일러가 primitive 타입에서 wrapper class로 자동 변환 시키는 것 

→ Object 객체들의 값들을 wrapper class로 만들어 null이 허용 가능하게 한다.

→ 하지만 명시적으로 박싱하는 경우 새로운 object를 생성한다. 

5**) hashcode를 지연 초기화하는 hashcode 메서드 (스레드 안정성 까지 고려)** 

```java
@Override
    public int hashCode(){
			int hash = hashCode; 
			if(result == 0){
				hash = address;
				hash = 31 * hash  +(city != null ? city.hashCode() : 0);
				hash = 31 * hash +(street != null ? street.hashCode() : 0);
      }
			return hash;
  }
```

위에 코드는 hashcode에 이미 계산된 해시코드를 저장하고 값이 없다면 처음 계산을 수행하고 캐시에 저장한다. 

이를 lazy loading이라고 한다. **처음에 계산이 실행되지 않고 필요한 시점에 한번만 계산한다.** 

캐싱은 해시 코드 계산 비용이 높거나 자주 호출되는 상황에서 성능을 향상시킬수 있다. 객체의 상태가 변경되지 않는 한 해시 코드가 변하지 않는다면 한 번 계산한 해시코드를 사용하는 것은 합리적이다. 하지만 캐싱된 해시코드의 상태가 변경되었다면 캐시를 갱신해줘야 한다. 

**1. 불변 객체에 대해 hashcode 생성비용이 많이 든다면,  캐싱을 고려해야합니다.**

이 때, 지연 초기화로 구현할 경우 스레드 안정성 또한 고려해야합니다.

**2. 성능을 높이기 위해  핵심필드를 제외하고 hashCode를 계산하지 않습니다.**

속도는 빨라져도 hash 품질이 나빠져 해시 테이블 성능을 떨어트릴 수 있습니다.

**3. Hashcode가 반환하는 값의  생성규칙 을 API 사용자에게 자세히 공표하지 않습니다.**

클라이언트가 hashcode값에 의지해 코드를 짜지 않기 위함입니다.
