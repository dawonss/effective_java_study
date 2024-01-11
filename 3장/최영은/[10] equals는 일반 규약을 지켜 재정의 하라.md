## 10. equals는 일반 규약을 지켜 재정의 하라
### 1️⃣ equals를 재정의 하지 않을 상황
1. 각 **인스턴스가 본질적으로 고유**한 경우
    * 값 표현이 아닌 **동작하는 개체를 표현하는 클래스**
    * Thread가 여기에 해당
    * Object의 equals 메서드는 이러한 클래스에 딱 맞게 구현됨
2. 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없는 경우
    * 설계자나 클라이언트가 **논리적 동치성을 검사할 필요가 없다**고 생각하거나 **원하지 않는 경우** Object.equals()만으로도 해결됨
3. 상위 클래스의 **재정의된 equals가 하위 클래스에도 딱** 들어 맞는 경우
    * ex. AbstractSet-Set, AbstractList-List, AbstractMap-Map
    * 상위 클래스인 AbstractOOO가 구현한 equals를 각 구현체들은 상속받아 그대로 사용
4. 클래스가 private 혹은 package-private이고 .equals()를 호출할 일이 없는 경우
### 2️⃣ equals를 재정의 해야하는 상황
1. 객체 식별성이 아닌 **논리적 동치성을 확인**해야하는데
2. **상위 클래스**의 equals가 **논리적 동치성을 비교하도록 재정의 되지 않은** 경우
    * ex. **값 클래스** : Integer, String과 같은 **값을 표현하는 클래스**들이 해당
    * 이 경우에 만일 재정의 해둔다면 **해당 인스턴스의 값을 비교할 수 있**는 것을 기본이고, **Map과 Set의 원소로도 사용할 수 있**게 된다.
* 단, 값 클래스 중 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 **인스턴스 통제 클래스**의 경우에는 **재정의 하지 않아도 된다. (Enum도 해당)**
* 이 경우, **논리적으로 같은 인스턴스가 2개 이상 만들어 질 수 없기 때문**에 **논리적 동치성과 객체 식별성이 동일한 의미**를 나타내게 된다. Object.equals() 만으로 논리적 동치성까지 확인이 가능
### 3️⃣ equals 재정의 시 따라야하는 일반 규약
1. **반사성**
     * null이 아닌 모든 참조값 x에 대해, **x.equals(x) ➡️ true**
3. **대칭성**
     * null이 아닌 모든 참조값 x, y에 대해, **x.equlas(y) == true ➡️ y.equals(x) == true**
5. **추이성**
     * null이 아닌 모든 참조값 x, y, z에 대해, **x.equlas(y) == true &&
y.equlas(z) == true ➡️ x.equlas(z) == true**
7. **일관성**
     * null이 아닌 모든 참조값 x,y에 대해, **x.equals(x)의 값이** 몇번을 출력하든 **항상 true거나 false**여야 한다
9. **null-아님**
      * null이 아닌 모든 참조값 x에 대해, **x.equlas(null) == false**
### 4️⃣ 반사성
* 객체는 **자기 자신과 같아야 한다**
### 5️⃣ 대칭성
* 두 객체는 **서로에 대한 동치 여부를 똑같이** 답해야 한다
```java
//잘못된 코드 : 대칭성 위배
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s){
        this.s = Objects.requireNonNull(s);
    }
    //대칭성 위배 : String도 포함
    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
        if (o instanceof String)
            return s.equalsIgnoreCase((String) o);
    }
    //변경된 메서드 : 오직 CaseInsensitiveString인지만 확인
    @Override public boolean equals(Object o) {
        return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s); 
    }
    ...
}

//사용 예제
public class Main{
    public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String s = "polish";
        cis.equals(s); //1. 반환 : true
        s.equals(cis); //2. 반환 : false

        List<CaseInsensitiveString> list = new ArrayList<>();
        list.add(cis); //3. 컬렉션에 넣는 경우
        list.contains(s);
    }
}
```
> 1. CaseInsensitiveString.equals()는 String을 알고있어 true로 반환
> 2. String.equals()는 CaseInsensitiveString을 알지못해 false로 반환
> 3. 구현하기 나름이라 equals 규약을 어긴객체를 사용하는 다른 객체들이 어떤 반응을 보일지 알 수 없다
> 4. 해결 : 따라서 CIS와 String을 연동하려 하지말고 CIS인지만 확인하도록 하라!
### 6️⃣ 추이성 
* **첫번째와 두번째** 객체가 같고 **두번째와 세번째 객체가 같다면**, **첫번째와 세번째 객체도 같아야** 한다
```java
//1. 상위 클래스 Point
public class Point{
   private final int x;
   private final int y;

   public Point(int x, int y) {
      this.x = x;
      this.y = y;
   }

   @Override public boolean equals(Object o) {
      if (!(o instanceof Point))   return false;
      Point p = (Point) o;
      return p.x == x && p.y == y;
   }
   ...
}
//2. 확장한 하위 클래스 ColorPoint : 새로운 필드인 color 추가
public class ColorPoint extends Point {
   private final Color color;
   public ColorPoint(int x, int y, Color color) {
      super(x, y);
      this.color = color;
   }
   ...
}
```
> 이 경우 Point.equals() 메서드에서는 ColorPoint의 주요 정보인 color를 비교할 수 없이 무시되기 때문에 만족할 수 없다
```java
//잘못된 코드1 - 대칭성 위배 : ColorPoint.equals()에서 ColorPoint 클래스 타입만 확인
public class ColorPoint {
   ...
   @Override public boolean equals(Object o) {
      if (!(o instanceof ColorPoint))   return false;
      return super.equals(o) && ((ColorPoint) o).color == color;
   }
}

//사용 예제
public class Main {
   public static void main(String[] args){
      Point p = new Point(3,4);
      ColorPoint cp = new ColorPoint(3,4,Color.BLUE);
      p.equals(cp);   //1. 반환 : true
      cp.equals(p);   //2. 반환 : false
   }
}
```
> 1. Point.equals()에서는 Point의 하위 클래스인 ColorPoint를 비교하여 true가 반환되지만 color 정보를 비교하지 않고 있음
> 2. ColorPoint가 아닌 Point 클래스를 매개변수로 받았기 때문에 false 반환
```java
//잘못된 코드2 - 대칭성은 맞지만, 추이성 위배
public class ColorPoint {
   ...
   @Override public boolean equals(Object o) {
      if (!(o instanceof Point))   return false;
      if (!(o instanceof ColorPoint))   return o.equals(this);
      return super.equals(o) && ((ColorPoint) o).color == color;
   }
}

//사용 예제
public class Main {
   public static void main(String[] args){
      Point p = new Point(3,4);
      ColorPoint cp1 = new ColorPoint(3,4,Color.BLUE);
      ColorPoint cp2 = new ColorPoint(3,4,Color.WHITE);
      cp1.equals(p);   //1. 반환 : true
      p.equals(cp2);   //2. 반환 : true
      cp1.equals(cp2);   //3. 반환 : false
   }
}
```
> 1. ColorPoint.equals() 내에 Point 클래스도 추가되어 true 반환
> 3. color 값이 다르기 때문에 false 반환 <br>
> * 또한 이 방식은 무한 재귀를 일으킬 가능성도 존재함
> * 사실상 **구체 클래스를 확장하여 새로운 값을 추가하면서 equals 일반 규약을 만족할 수는 없음**!
> * * 자바 라이브러리의 java.sql.Timestamp는 java.util.Date를 확장한 후 nanoseconds 필드를 추가하였고, 결과적으로 Timestamp.equals()의 대칭성 위반 문제로 Timestamp와 Date를 섞어서 사용하는 경우 문제가 될 수 있다고 함 주의!
```java
//우회 방법 : equals 일반 규약 지키면서 값 추가 성공
public class ColorPoint {
   private final Point point;   //1. Point 클래스 타입의 필드 추가
   private final Color color;

   public ColorPoint(int x, int y, Color color) {
      point = new Point(x, y);
      this.color = Objects.requireNonNull(color);
   }

   public Point asPoint() {   //2. point 조회 시 이용할 메서드 추가
      return point;
   }

   @Override public boolean equals(Object o) {   //3. point와 color 모두 비교 가능
      if (!(o instanceof ColorPoint))   return false;
      ColorPoint cp = (ColorPoint) o;
      return cp.point.equals(point) && cp.color.equals(color);
   }
   ...
}
```
> 1. 상속 대신 컴포지션을 사용하여 상위 클래스 타입의 필드를 private으로 둔다
> 2. 상위 클래스 타입의 필드를 조회할 수 있는 public 메서드를 추가한다
> 3. equals() 내에서 point와 color 필드를 각각 비교하도록 한다
### 7️⃣ 일관성
* **두 객체가 같다면 수정되지 않는 한 영원히 같아야** 한다
    * 그러므로 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다
    * 항시 메모리에 존재하는 객체만 사용하여 계산을 수행 해야 한다
    * 예) java.net.URL의 equals
### 8️⃣ null-아님
* **모든 객체가 null과 같지 않아야** 한다
* 값이 true를 반환하는 것이 아니더라도 NullPointerException을 던지는 것 조차 안된다
* 간혹 **null 값도 정상값으로 취급하는 참조 타입 필드**도 있는데 이런 필드는 **Object.equals(Object, Object)로 비교**하여 NullPointException을 예방하도록 한다
```java
//1. 명시적 null 검사 : 입력이 null인지 확인. 이러한 검사는 필요 없음
@Override public boolean equals(Object o) {
   if (o == null)   return false;
   ...
}

//2. 묵시적 null 검사 : 이경우가 조금 더 나음.
@Override public boolean equals(Object o) {
   if (!(o instanceof MyType))   return false;
   MyType mt = (MyType) o;
   ...
}
```
> 동치성을 검사하기 위해서는 적절히 형변환하여 필수 필드 값을 비교해야하므로 instanceof를 이용하여 올바른 타입인지 확인 <br>
> 만일 타입을 확인하지 않아 잘못된 타입이 인수로 주어진다면 ClassCastException을 던져 일반 규약을 위배하게 됨 <br>
> instanceof는 비교 대상으로 null이 주어져도 false를 반환하기 때문에 명시적인 null 검사는 불필요 <br>
### 9️⃣ 양질의 equals 메서드 구현 방법
1. **== 연산자를 사용**해 입력이 **자기 자신의 참조인지 확인**하여 자기 자신이라면 **true 반환**
2. **instanceOf 연산자로 입력이 올바른 타입인지 확인**하여 그렇지 않다면 false 반환
    * 보통 정의된 클래스가 올바른 타입
    * 하지만 만일 인터페이스가 필요에 의해 자신을 구현한 서로 다른 클래스간의 비교를 원한다면, 이를 구현한 클래스들도 equals에서 인터페이스를 사용하여야 한다
         * ex) Set, List, Map 등 컬렉션 인터페이스
3. 입력을 올바른 타입으로 **형변환**
4. 입력 객체와 자기 자신의 대응되는 **'핵심' 필드 들이 모두 일치하는지 하나씩 검사**
    * 하나라도 다르면 false, 모두 일치하면 true 반환
    * 2단계에서 인터페이스를 사용한 경우 입력 객체의 필드 값을 가져오는 경우에도 해당 인터페이스의 메서드를 사용하여야 한다.
### 0️⃣ 그 외 주의사항
1. float와 double은 .compare(), 이를 제외한 기본 타입 필드는 ==, 참조 타입 필드는 각 equals()로 비교한다
2. 배열 필드는 각 원소를 비교하고, 배열의 모든 원소가 핵심 필드인 경우 Arrays.equals() 중 하나를 사용한다
3. 불변이며 비교가 복잡한 필드를 가진 클래스의 경우 필드의 표준형을 저장하여 표준형끼리 비교하도록 한다
4. 필드는 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하도록 한다
5. **equals()를 재정의 할 땐 hashCode도 반드시 재정의 한다**
6. equals()를 재정의 할 땐 너무 복잡하게 비교하기 보단 **필드들의 동치성만 검사한다고 생각하고 접근하자**
7. Object.equals()를 재정의 하는 것으로 **equals()의 매개변수는 반드시 Object 타입**으로 받는다
8. AutoValue를 이용하면 자동으로 테스트 할 수 있다
### ❗ 핵심 요약
1. **꼭 필요한 경우가 아니면 equals()를 재정의 하지 않는다**
2. 재정의 하는 경우 클래스 내의 **모든 핵심 필드를 equals 일반 규약에 맞게 비교**하도록 한다
