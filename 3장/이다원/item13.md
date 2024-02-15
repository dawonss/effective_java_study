# 아이템13- clone 재정의는 주의해서 진행하라

## Cloneable 인터페이스

Cloneable은 Object에 정의되어 있는 clone을 사용하기 위한 인터페이스이다. 인터페이스를 내부를 살펴보면 clone 메서드가 선언되어 있지 않고, **Object 클래스에 clone 메서드가 protected로 선언되어 있다.** 

Cloneable 인터페이스는 Object의 클래스의 clone 메서드가 어떻게 동작할 것인지 결정한다. **Cloneable을 구현한 인스턴스에서 clone을 호출하면 그 객체의 필드 단위로 복사한 객체를 반환한다.** 반대로 Cloneable을 구현하지 않으면  CloneNotSupportedException 예외를 던진다. 

### clone 메소드는 어디서 정의되는 것일까?

clone은 Object 클래스에 정의되어 있다. 객체가 clone 메서드를 상속해도 clone 메서드의 접근제어자는 protected 이므로 메서드를 호출할 수 없다. 

### clone 메서드를 재정의해야 하는가?

clone을 사용하기 위해서 clonable 인터페이스를 구현해야 한다. 또한 clone  메서드의 접근제어자를 protected 에서 public으로 변경해야한다. 

그렇지 않으면 다른 클래스에서 clone 메서드를 호출할 수 없다. 

Object의 clone 메서드는 clone를 반환하지만 우리가 사용할 객체의 clone 메서드는 해당 객체를 반환하게 해야 한다. 이는 자바가 곧ㅇ변 반환 타이핑을 지원하기 때문이다. 

`공변 반환 타입`이란?

- 부모 클래스의 메소드를 오버라이딩 하는 경우, 부모 클래스의 반환 타입은 자식 클래스의 타입으로 변경이 가능하다.
- Object의 메소드인 clone을 오버라이딩 하였기 떄문에, phoneNumber의 clone 메서드는 super.clone에서 얻은 객체를 반환하기 전에 PhoneNumber로 형변환 하였다.

## clone 규약

- x.clone() ≠ x 반드시 true (객체의 논리적인 동치성이 아니라 레퍼런스 자체가 다른 오브젝트를 참조해야 한다.)
- x.clone().getClass() == x.getClass() 반드시 true
- x.clone().equals(x) true가 아닐 수도 있다.

불변 객체라면 다음으로 충분하다. 

- Cloneable 인터페이스를 구현하고
- clone 메서드를 재정의한다. 이때 super.clone()을 사용한다

### clone 메소드의 문제점

가변 객체가 포함된 클래스를 복사하는 경우에는 super.clone() 결과를 그대로 반환하면 예상치 못한 곳에서 오류가 발생할 수 있다. 

구체 클래스가 가변 객체를 참조할 때, 원본과 복제본 객체는 동일한 가변 객체를 참조하고 있기 때문에 어느 한 곳에서 수정이 일어나면 다른 곳도 영향을 받게 되어 불변식을 해친다. 

### clone 메서드는 사실상 생성자와 같은 효과를 낸다.

clone 메서드는 사실상 생성자와 같은 효과를 낸다. 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다. 

배열의 clone은 런타입 타입과 컴파일 타입 모두가 원본 배열과 똑같은 배열을 반환한다. 따라서 배열을 복제할 때는 배열의 clone 메서드를 사용하라고 권장한다. 

```java
@Override public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```

하지만 clone은 재귀적으로 호출하는 것만으로 충분하지 않을 수도 있다. 

해시테이블 clone 메서드를 생각해보면 해시테이블 내부는 버킷들의 배열이고, 각 버킷은 키-값 쌍을 담는 연결리스트의 첫번째 엔트리를 참조한다.

 

```java
public class HashTable implements Cloneable {
	private Entry[] buckets = ...;
    private static class Entry {
	    final Object key;
    	Object value;
        Entry next;

		Entry(Object key, Object value, Entry next) {
        	this.key = key;
            this.value = value;
            this.next = next;
		}
        ... // 나머지 코드는 생략
	}

    @Override
    public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = buckets.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
```

복제본은 자기만의 버킷 배열을 갖지만, 이 배열은 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않게 동작할 가능성이 생긴다. 

이를 해결하려면 각 버킷을 구성하는 연결리스트를 복사해야 한다.

```java
public class HashTable implements Cloneable {
	...

	Entry deepCopy() {
    	return new Entry(key, value, next == null ? null : next.deepCopy());
    }

	@Override
    public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            for (int i = 0; i < buckets.length; i++)
                if (buckets[i] != null)
                    result.buckets[i] = buckets[i].deepCopy();
                return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
	}
    
}
```

 

private 클래스인 HashTable.Entry는 깊은 복사를 지원하도록 보강되었다. 

HashTable의 clone 메서드는 먼저 적절한 크기의 새로운 버킷 배열을 할당한 다음 원래의 버킷 배열을 순회하며 비지 않은 각 버킷에 대해 깊은 복사를 수행한다. 

이때 Entry deepCopy 메서드는 자신이 가르키는 연결 리스트 전체를 복사하기 위해 자신을 재귀적으로 호출한다. 

Cloneable을 구현하는 모든 클래스는 clone을 재정의 해야한다. 이때 접근제어자는 public으로, 반환 타입은 클래스 자신으로 변경한다. 이 메서드는 가장 먼저 super.clone을 호출한 후 필요한 필드를 전부 적절히 수정한다.

하지만 기본 타입 필드와 불변 객체 참조만 갖는 클래스라면 아무필드도 수정할 필요가 없다. 단, 일련번호나 고유 ID는 비록 기본 타입이나 불변일지라도 수정해줘야 한다.
