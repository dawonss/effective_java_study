# 아이템12- toString을 항상 재정의하라

Object의 기본 toString 메서드가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없다. 

이 메서드는 단순히 `클래스_이름@16진수로_표현한_해시코드` 를 반환할 뿐이다.

Object에서 기본적으로 제공하는 toString같은 경우에는 클래스이름뒤에 16진수로 표현한 해쉬코드를 반환한다.

```java
PhoneNumber jenny = new PhoneNumber(707, 867, 5309);
System.out.println("제니의 번호: " + jenny);
```

**실행결과**

```
me.whiteship.chapter02.item12.PhoneNumber@adbbd
```

위와 같은 형태는 유용하지 않다. 

따라서 toString의 일반 규약에 따르면 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다. 

toString을 잘 구현한 클래스는 사용하기 훨씬 즐겁고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다.  

### toString 구현할 때 반환값의 포맷을 문서화할지 정해야한다.

toString을 구현할 때면 반환값의 포맷을 문서화할지 정해야 한다. 

전화번호나 행렬 같은 값 클래스라면 문서화하기를 권한다. 

포맷을 명시하기로 했다면, 명시한 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩터리나 생성자를 함께 제공해주면  좋다. 

**포맷을 명시하려면 아주 정확하게 해야한다.** 

```java
/**
     * 이 전화번호의 문자열 표현을 반환한다.
     * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
     * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
     * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
     *
     * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
     * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
     * 전화번호의 마지막 네 문자는 "0123"이 된다.
     */
    @Override public String toString() {
        return String.format("%03d-%03d-%04d",
                areaCode, prefix, lineNum);
    }
```

toString에 재정의 되어 있는 것처럼 areaCode, prefix, lineNum을 전달받는 방법이 따로 있어야 한다. 따라서 위의 정보를 받을 수 있도록 정적팩토리 메서드를 추가할 수 있어야 한다. 

### 정적 팩토리 메서드 추가

```java
public static PhoneNumber of(String phoneNumberString) {
        String[] split = phoneNumberString.split("-");
        PhoneNumber phoneNumber = new PhoneNumber(
                Short.parseShort(split[0]),
                Short.parseShort(split[1]),
                Short.parseShort(split[2]));
        return phoneNumber;
    }
```

정적팩토리 메서드를 생성해서 나중에 사용자가 받은 문자열을 기반으로 

phoneNumber라는 인스턴스를 만들 수 있도록 구현한다.
