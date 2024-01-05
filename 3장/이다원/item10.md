# 아이템10 - equals는 일반 규약을 지켜 재정의하라

## **equals()란?**

**equals()는 두 객체가 논리적으로 동등한지 여부를 확인**

**동등성과 동일성을 구분해보자!**

**`동등성` :**  객체의 주소값이 다르더라도 내용이 같다면 같다고 보는 것. ex) equals()

**`동일성` :** 객체의 주소값이 다르면 아무리 같은 내용이라도 같지 않다고 보는 것. 즉, 메모리에 저장된 주소 공간이 같은 경우에는 객체 A와 B의 값이 같다. 

 ex) ==  

1. **참조의 동등성 (Reference Equality):** 두 참조 변수가 동일한 객체를 가리키고 있는지 여부를 확인합니다. 즉, 두 참조의 값(주소)이 같은지를 검사합니다. 이는 **`==`** 연산자를 사용하여 확인됩니다.
    
    ```java
    Object obj1 = new Object();
    Object obj2 = obj1; // obj2는 obj1을 가리킴
    
    System.out.println(obj1 == obj2); // true
    ```
    
2. **객체의 동등성 (Object Equality):** 두 객체의 내용이 같은지를 비교합니다. 이는 객체가 같은 내용을 가지는지 여부에 따라 결정됩니다. 주로 **`equals`** 메서드를 사용하여 객체의 동등성을 비교합니다.
    
    ```java
    String str1 = new String("Hello");
    String str2 = new String("Hello");
    
    System.out.println(str1.equals(str2)); // true (문자열 내용이 같음)
    ```
    

객체의 동등성은 객체의 내용이 동일한지를 판단하는 데 사용되므로, **`equals`** 메서드를 적절하게 구현하여 동등성을 정의하는 것이 중요하다. 

만약 클래스에서 **`equals`** 메서드를 오버라이드하지 않는다면, 기본적으로는 참조의 동등성이 비교된다.

### equals를 재정의 하지 말아야 하는 이유

1. **각 인스턴스가 본질적으로 고유하다.** 
    
    값이 아닌 `Thread` 와 같이 동작하는 개체를 표현하는 클래스라면 재정의할 필요없다.  
    
2. **인스턴스의 ‘논리적 동치성(logical equality)’를 검사할 일이 없는 경우** 
    
    java.util.regex.Pattern의 equals를 재정의해서 두 pattern의 인스턴스가 같은 정규 표현식을 나타내는지를 검사하는 방법이다. 
    
3. **상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어 맞는 경우** 
- `Set` 구현체 : `AbstractSet`이 구현한 equals를 상속
- `List` 구현체 : `AbstractList`이 구현한 equals를 상속
- `MAP` 구현체 : `AbstractMap`이 구현한 equals를 상속
1. **클래스가 private나 package-private이며, equals 메서드 호출할 일이 없는 경우**
2. 
    
    equals의 호출을 막아주면 좋다 
    
    ```java
    @Override public boolean equals(Object O) {
    	throw new AssertionError();   //호출 금지
    }
    ```
    
    ### equals를 재정의 해야하는 경우
    
    - 객체 식별성이 아니라 논리적 동치성을 확인해야 한다.
    - 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때이다.
    - equals 메서드를 재정의할 때 반드시 Object 명세 규약에 따라야 한다.
    - Object 명세서에서 말하는 동치관계? → equals 메서드가 쓸모있으려면 모든 원소가 같은 동치류에 속한 어떤 원소라도 교환 가능해야 한다.
    
    ### 1. 반사성(reflexivity)
    
    객체는 자기 자신과 같아야 한다는 뜻이다. 
    
    ### 2. 대칭성(symmetry)
    
    ```java
    x.equals(y) == y.equals(x)
    ```
    
    x.equals(y)가 참이면 그 반대 y.equals(x)도 참이여야 한다. 
    
    즉, 두 객체는 서로에 대한 동치 여부에 대한 결과가 같아야 한다. 
    
    ### 3. 추이성(transitivity)
    
    ```java
    x.equals(y) == true
    y.equals(z) == true
    x.equals(z) == true
    ```
    
    첫 번째 객체와 두 번째 객체가 같고, 두번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다는 뜻이다. 
    
    **상위클래스에 없는 새로운 필드를 하위클래스에 추가하는 상황에서 어기기 쉽다. 어떤 방식을 해도 상속을 통해서 equals 조건을 충족시킬 수 없다.** 
    
    **대칭성 위배**
    
    ```java
    // 첫 번째 equals 메서드(코드 10-2)는 대칭성을 위배한다. (57쪽)
            Point p = new Point(1, 2);
            ColorPoint cp = new ColorPoint(1, 2, Color.RED);
            System.out.println(p.equals(cp) + " " + cp.equals(p));
    ```
    
    **→  Result: true false**
    
    메서드에서 `Point` 와 `ColorPoint` 를 바꿔서 비교해보면, 결과값이 다를 수 있다(대칭성 위배). 
    
    `p.equals(cp)` 는 색상 필드를 무시하고, `cp.equals(p)` 는 입력 매개변수의 클래스 종류가 달라 `instanceof` 를 통과하지 못하기 때문이다.
    
    **추이성 위배**
    
    ```java
    // 두 번째 equals 메서드(코드 10-3)는 추이성을 위배한다. (57쪽)
            ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
            Point p2 = new Point(1, 2);
            ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
            System.out.printf("%s %s %s%n",
                              p1.equals(p2), p2.equals(p3), p1.equals(p3));
    ```
    
    **→  Result: false true false**
    
    `p1.equals(p2)`와 `p2.equals(p3)`는 색상을 무시하여 `true`를 반환하지만, `p1.equals(p3)`는 색상을 고려하게 되어 `false` 를 반환하기 때문이다. 둘의 결과값이 다르므로 추이성에 위반된다.
    
    **리스코프 치환 원칙 위배**
    
    얼핏 객체 지향 추상화인, `instanceof` 검사를 `getClass` 검사로 바꾸면 규약도 지키고 값도 추가하면서 구체 클래스를 상속할 수 있다는 뜻으로 들린다. 그러나 이는 리스코프 치환 원칙을 위반한다. 
    
    **리스코프 치환 원칙(Liskov substitution principle)**
    
    상위 타입의 자료형은 하위 타입으로 변환되어도 작동해야 한다)
    
    ```java
    public class Point{
      @Override public boolean equals(Object o) {
         if (o == null || o.getClass() != getClass())
           return false;
         Point p = (Point) o;
         return p.x == x && p.y == y;
      }
    }
    ```
    
    위의 코드는 같은 구현 클래스 객체와 비교할 때만 true를 반환한다. 
    
    즉, point의 하위 클래스를 비교할 때는 항상 false를 반환하게 된다. 
    
    아쉽게도 구체클래스를 확장해 새로운 값을 추가하면서 equals 규약을 
    
    만족시킬 방법은 존재하지 않는다. 
    
    하지만 상속 대신 컴포지션을 사용하라(아이템 18)을 이용하면 된다. 
    
    즉, point를 상속하는 대신 `private`필드로 두고, 일반 point를 반환하는 뷰 메서드를 추가하는 방식이다. 이 방식은 `equals`의 대칭성, 추이성을 지키면서 값을 추가할 수 있다. 
    
    ```java
    public class ColorPoint {
        private final Point point;  // 컴포지션
        private final Color color;
    
        public ColorPoint(int x, int y, Color color) {
            point = new Point(x, y);
            this.color = Objects.requireNonNull(color);
        }
    
        public Point asPoint() {  // 뷰 반환 메서드
            return point;
        }
    
        @Override public boolean equals(Object o) {
            if (!(o instanceof ColorPoint))
                return false;
            ColorPoint cp = (ColorPoint) o;
            return cp.point.equals(point) && cp.color.equals(color);
        }
    
        @Override public int hashCode() {
            return 31 * point.hashCode() + color.hashCode();
        }
    }
    ```
    
    ### 4. 일관성(consistency)
    
    - 불변 객체는 한번 같으면 끝까지 같아야 하고, 다르면 끝까지 달라야 한다.
    - `equals` 의 판단에 신뢰할 수 없는 자원이 끼워들게 해서는 안된다.
    - `equals`는 항상 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다
    
    ### 5. null - 아님
    
    - 모든 객체가 null과 같지 않아야 한다.
    - `NullPointerException` 도 발생시키지 않아야 한다.
    
    ## 양질의 equals를 구현하는 단계
    
    1. **== 연산자를 사용해 입력이 자기 자신의 참조인지 확인하라.** 
        
        자기 자신이면 true를 반환한다. 단순 성능 최적화용 
        
    2. **instanceof 연산자로 입력이 올바른 타입인지 확인한다.** 
        
        같은 인터페이스를 구현한 클래스끼리 비교하고 싶을 땐 equals에서 인터페이스를 사용해야 한다. 
        
        ex) `List`, `Map`, `Map.Entry`  
        
    3. **입력을 올바른 타입으로 형변환 한다.**
    4. **입력 객체와 자기 자신의 대응되는 ‘핵심’ 필드들이 모두 일치하는지 하나씩 검사한다.** 
    - float, double을 제외한 기본타입은 == 비교한다.
    - `float`과 `double`필드는 각각 정적 메서드인 `Float.compare(float, float)`과 `Double.compare(double, double)` 로 비교한다.
    

## 주의사항

- equals를 재정의할 때 hashcode도 반드시 재정의 하자
- 필드들의 동치성만 검사하고 너무 욕심부려서 비교하려 하지 말자
- 매개변수의 타입을 Object로 하라
