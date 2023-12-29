## 12. toString을 항상 재정의 하라
* Object.toString()은 단순히 **클래스_이름@16진수로_표시한_해시코드** 를 반환한다 <br>
* toString() 일반 규약
    * '**간결하면서 사람이 읽기 쉬운 형태의 유익한' 정보**를 반환 <br>
        * ex. PhoneNumber 클래스의 경우, PhoneNumber@adbbd 보다 707-1234-5678과 같은 번호를 알려주는 게 더 유익한 정보이다 <br>
    * **모든 하위 클래스에서 이 메서드를 재정의** 하라
### 1️⃣ toString()을 잘 구현했을 때의 이점
* 사용하기 좋고, 사용한 시스템의 디버깅이 쉽다
* toString()이 직접 호출하지 않아도 어디선가 **자동으로 불리는 경우**
    1. 객체를 참조하는 컴포넌트가 **오류 메시지를 로깅**할 때
    2. **println, printf, 문자열 시스템 연산자(+), assert 구문**에 객체를 넘길 때
    3. **디버거가 객체를 출력**할 때
* 좋은 toString()은 이 인스턴스를 포함하는 객체에서 유용하게 쓰일 수 있다
    * ex) map 객체 출력 시, {Jenny=PhoneNumber@adbbd} 보다 {Jenny=070-1234-5678}이 더 나음!
### 2️⃣ 필수 사항
* 실전에서 toString()은 **그 객체가 가진 주요 정보를 반환하는 게 좋다**
    * but, 객체가 거대하거나 객체 상태가 문자열로 표현하기에 적합하지 않다면 요약 정보를 담도록 한다
### 3️⃣ toString() 구현 시 반환 값 포맷의 문서화
1. **장점**
    * **표준적**이고 **명확**하고 **사람이 읽을 수 있는 데이터 객체로 저장할 수 있게 됨**
    * 명시화 하는 경우 포맷에 맞는 **문자열과 객체를 상호 전환 할 수 있는 팩터리 메서드 혹은 생성자를 함께 제공해주면 좋다**
        * ex. BigInteger, BigDecimal, 기본 클래스들
2. **단점**
    * **한 번 명시하면 평생 얽매이게 될 수 있음**
    * 프로그래머들은 그 포맷에 맞춰 파싱하고, 새 객체를 생성하고, 영속 데이터로 저장하는 코드를 작성할 것 임!
    * 만일 추후에 포맷이 바뀐다면 이전 코드와 데이터들이 엉망이 됨
    * 명시 하지 않으면 포맷을 수정 및 개선 할 수 있는 유연성을 얻게 됨
3. 포맷을 명시하든 아니든 **의도는 명확히 밝혀야 한다**
4. 포맷을 명시하든 아니든 **toString() 반환 값에 포함된 정보를 가져올 수 있는 API를 제공해야 한다**
    * **제공하지 않으면 toString()의 반환값을 파싱하여 사용**해야된다
        * 이는 성능 저하를 일으키고 불필요한 작업이 된다.
        * 이렇게 되면 포맷이 준-표준 API와 다름이 없어진다
    * 향후 **포맷을 바꾸면 시스템이 망가지게 될 수 있다**
### 4️⃣ 필요한 경우와 그렇지 않은 경우
1. **필요한 경우**
    * 하위 클래스들이 공유해야할 문자열 표현이 있는 추상 클래스
2. **필요하지 않은 경우**
    * **정적 유틸리티 클래스**는 toString을 제공할 이유가 없음
    * **열거타입**은 자바에서 이미 완벽한 **toString을 제공**해줌
### 5️⃣ AutoValue 프레임워크
* AutoValue가 toString도 생성해줌 하지만 이 toString은 각 필드의 내용은 나타나지만 의미는 파악할 수 없음
* 그래도 Object.toString() 보다 자동생성된 toString()이 유용하다
### ❗ 핵심 요약
1. **모든 구체 클래스에서 Object.toString()은 재정의** 하라. 상위 클래스에 이미 알맞게 정의 된 경우는 예외
2. toString()은 **객체에 대한 명확하고 유용한 정보를 읽기 좋은 형태로 반환**해야 한다.

### 👩🏻‍💻 실습 코드
   ```java
   // enum 코드
   public enum UserStatus {
	   USER_SIGNUP("signup"),
	   USER_LOGIN("login"),
	   USER_LOGOUT("logout"),
	   USER_WITHDRAW("withdraw");
	   ...
   }
   // service 계층 코드
   public UserInfoResponseDto getInformation(String email) {
	     ...
        // toString()을 호출해도 되지만 println으로 출력하여 자동으로 toString()이 호출
        System.out.println("User's toString : "+user);  //User는 class 타입
        System.out.println("User.Status's toString : "+user.getStatus());  //User.Status는 enum 타입
        ...
   }
   ```
   ![결과](https://github.com/Choi-Young-Eun/effective_java_study/assets/71817230/94e452ad-88a1-4e65-be56-702465d0458d)
