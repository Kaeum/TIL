# 이펙티브자바 2장 - 객체 생성과 파괴

## 아이템1 생성자 대신 정적 팩터리 메서드를 고려하라

* 클라이언트가 클래스 인스턴스를 만드는 전통적인 방법은 public 생성자를 이용하는 것.   
꼭 알아둬야 할 다른 한 가지 기법이 `static factory method`   
아래는 그 예시.
  ```java
  public static Boolean valueOf(boolean b){
      return b ? Boolean.TRUE : Boolean.FALSE;
  }
  ```
***
* `static factory method`의 장점
    1. 이름을 가질 수 있다.
        + 반환될 객체의 특성을 설명하는 데 생성자보다 더 효과적이다.
        + `BigInteger(int, int, Random)` vs `BigInteger.probablePrime`
        + 하나의 시그니처로는 하나의 생성자만 만들 수 있다.
    2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
        + 불변 클래스[아이템17]는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스 캐싱하여 재활용.
        + 생성 비용이 큰 객체가 자주 요청되는 상황에 유리. (`Flyweight pattern`과 비슷한 기법)
    3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
        + 다형성을 가질 수 있다는 말?
        + 유연함을 가진다(???)
    4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
        + `EnumSet`의 경우 내부적으로 매개변수가 64개 이하면 `RegularEnumSet`을, 그 이상이면 `JumboEnumSet`을 반환한다.
    5. 정적 팩토리 메소드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
        + `JDBC`의 경우 `getConnection()` 안에 들어갈 소스경로(문자열)에 따라서 반환되는 결과가 달라진다.
        + 그런데 `getConnection()` 함수를 작성하는 시점에는 `OracleDriver`나 `MysqlDriver` 등이 존재하지 않아도 된다는 내용인듯(?)
***
* `static factory method`의 단점
    1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다.
        + `Collections`의 유틸리티 클래스들은 static이므로, 상속할 수 없다.
        + 이 제약은 상속보다 컴포지션을 사용[아이템18]하도록 유도하므로 오히려 장점(???)
    2. 정적 팩토리 메서드는 프로그래머가 찾기 어렵다.
        + API 문서를 잘 써놓고 메서드 이름도 널리 알려진 규약을 따라 짓는 것을 지향해야 한다.

            
