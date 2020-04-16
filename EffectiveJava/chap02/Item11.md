# 이펙티브자바 3장 - 모든 객체의 공통 메서드

## 아이템11 equals를 재정의하려거든 hashCode도 재정의하라

* equals를 재정의한 클래스에서 hashCode를 재정의하지 않으면, 인스턴스를 `HashMap`이나 `HashSet` 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것.

* Object 명세에서 발췌한 규약
    + equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 `hashCode()`를 몇 번 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
    + equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 `hashCode()`는 똑같은 값을 반환해야 한다.
    + equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 `hashCode()`가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

* 위 규약에서 크게 문제가 되는 조항은 두 번째. **논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.**
    + equals는 물리적으로 다른 두 객체를 논리적으로 같다고 할 수 있으나, `Object`의 기본 `hashCode()`는 이 둘이 전혀 다르다고 판단

    ```java
    Map<PhoneNumber, String> m = new HashMap<>();
    m.put(new PhoneNumber(707, 867, 5309), "제니");
    ```

    + `PhoneNumber` 클래스는 아이템 10에서 활용했던 클래스. 이제 여기서 `m.get(new PhoneNumber(707, 867, 5309))`를 실행하면? "제니"가 아닌 `null`을 리턴한다.  여기에는 2개의 `PhoneNumber` 인스턴스가 사용됐는데, 하나는 `HashMap`에 "제니"를 넣을 때 사용됐고 (논리적 동치인) 다른 하나는 이를 꺼내려할 때 사용됐다.
    + `PhoneNumber` 클래스는 `hashCode()`를 재정의하지 않았기 때문에, 논리적 동치인 두 객체가 서로 다른 해시코드를 반환하여 두번째 규약을 어기는 것.

* 그렇다면 올바른 `hashCode()`는 어떻게 구현해야 하나?
    1. int 변수 result를 선언한 후 값 c로 초기화한다. 이때 c는 해당 객체의 첫번째 핵심 필드를 단계 2.a 방식으로 계산한 해시코드다.(핵심 필드란 equals 비교에 사용되는 필드)
    2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
        + a. 해당 필드의 해시코드 c를 계산한다.
            * i. 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스다. ex) int형 변수의 경우 `Integer.hashCode()`
            * ii. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 `hashCode()`를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면, 이 필드의 표준형(canonical representation)을 만들어 그 표준형의 `hashCode()`를 호출한다. 필드의 값이 `null`이면 0을 사용한다.(다른 상수도 괜찮지만 일종의 컨벤션)
            * iii. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.b 방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0을 추천한다)를 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashCode()를 사용한다.
        + b. 단계 2.a에서 계산한 해시코드 c로 result를 갱신한다. 코드로는 다음과 같다.
        ```java
        result = 31 * result + c;
        ```
    3. result를 반환한다.

(라고 장황하게 나와있으나, 이걸 직접 구현하는건 너무 비효율적이라고 생각한다. 그냥 *AutoValue* 같은 프레임워크를 쓰자...)

* 파생 필드(다른 필드로부터 계산할 수 있는 필드)는 제외해도 좋으며, equals 비교에 사용되지 않은 필드는 **반드시** 제거해야 한다. 그렇지 않으면 두번째 규약을 어기게 된다. <br/><br/>
***

```java
@Override public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

위의 코드는 `PhoneNumber` 클래스에 딱 맞게 구현된 `hashCode()`. 품질 면에서나 기능 면에서나 우수하다. 단, 해시 충돌이 더욱 더 적은 방법을 꼭 써야한다면 구아바의 `com.google.common.hash.Hashing`을 참고하자.
***

```java
@Override public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```

이번에는 `Objects` 클래스의 `hash()`를 사용해 구현한 코드. 그러나 성능이 아쉽기 때문에, 성능에 민감하지 않은 상황에서만 사용할 것.

***

```java
private int hashCode; // 0으로 초기화.

@Override public int hashCode() {
    int result = hashCode;
    if (result == 0){
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
```
지연 초기화(lazy initialization). 해시코드 계산 비용이 크고 해시의 키로 사용되지 않는 경우 고려할만 하다. 단, 이 경우 `PhoneNumber`클래스를 thread-safe하게 만들도록 신경써야 한다.   
*(만약 계산 비용도 크고 클래스가 불변이라면, 캐싱하는 방식을 고려해야 한다. 해시의 키로 사용될 것 같은 경우, 인스턴스가 만들어질 때 해시코드를 계산해둬야 한다.)*

    결론 : equals를 재정의할 때는 hashCode도 반드시 재정의하자. 단, AutoValue를 사용하여 자동화하는 것을 고려하자..