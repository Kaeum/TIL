# 이펙티브자바 2장 - 객체 생성과 파괴

## 아이템6 불필요한 객체 생성을 피하라

* 똑같은 기능의 객체를 매번 사용하기보다는, 재사용하는 편이 나을 때가 많다. 아래 예시를 보자.
    ```java
    String s = new String("bikini");
    ```
    해당 라인이 실행될 때마다 String 인스턴스를 새로 만들어 낸다. 아래와 같이 바꾼다면 이 문제를 개선할 수 있다.
    ```java
    String s = "bikini";
    ```
    문자열 리터럴을 사용할 경우, heap 메모리에 새로운 인스턴스를 만드는 대신 하나의 String 인스턴스를 사용한다.

* 정적 팩터리 메서드를 제공하는 불변클래스에서는 불필요한 객체 생성을 피할 수 있다.   
        `Boolean(String)` 생성자 대신 `Boolean.valudOf(String)` 팩터리 메서드를 사용하는 것이 좋다.   
        전자는 호출될 때마다 새로운 객체를 만드는 반면 후자는 객체를 캐싱하여 재사용한다.

* 생성 비용이 비싼 객체가 반복해서 사용된다면 캐싱하여 재사용하는 것이 좋다.   
   
      

    문자열이 유효한 로마 숫자인지를 확인하는 메서드를 작성한다고 해보자.
    ```java
    static boolean isRomanNumeral(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3}" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }
    ```
    *위와 같이 `String.matches`를 사용하는 방식은 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복해 사용하기엔 적합하지 않다.* 이 메서드가 내부에서 만드는 (정규표현식을 위한) `Pattern` 인스턴스는 한번 쓰고 버려져서 곧바로 GC의 대상이 된다. 심지어 `Pattern` 인스턴스는 생성 비용이 높다.

    ```java
    public class RomanNumerals {
        private static final Pattern ROMAN = Pattern.compile(
                                        "^(?=.)M*(C[MD]|D?C{0,3}" 
                                        + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

        static boolean isRomanNumeral(String s) {
            return ROMAN.matcher(s).matches();
        }                                        
    }
    ```
    생성 비용이 큰 `Pattern` 인스턴스를 클래스 초기화(정적 초기화) 과정에서 직접 생성해 캐싱해두고, 나중에 `isRomanNumeral` 메서드가 호출될 때마다 재사용한다.

* 어댑터(뷰)   

    `Map` 인터페이스의 `keySet` 메서드는 키 전부를 담은 `Set` 뷰를 반환한다. 무슨 말이냐면, `keySet`을 호출할 때마다 새로운 `Set` 인스턴스가 만들어지는 것이 아니라 매번 같은 `Set` 인스턴스를 반환한다. 이 때 반환된 `Set` 인스턴스는 불변이 아닌 가변이지만 모두 똑같은 기능을 한다. 따라서 `keySet`이 뷰 객체를 여러 개 만들어도 상관은 없지만, 그럴 필요도 없고 이득도 없다.

    ```java
    Map<String, Integer> serviceSinceMap = new HashMap<>();
    serviceSinceMap.put("Kakao", 2010); 
    serviceSinceMap.put("Naver", 1999);

    Set<String> test1 = serviceSinceMap.keySet(); 
    Set<String> test2 = serviceSinceMap.keySet();

    test1.remove("Kakao"); 
    System.out.println(test1 == test2); // true 
    System.out.println(test1.size()); // 1 
    System.out.println(test2.size()); // 1 
    System.out.println(serviceSinceMap.size()); // 1
    ``` 
    출처 : 백기선님 포스팅 참조
* 오토박싱
    ```java
    private static long sum() {
        Long sum = 0L;
        for(long i = 0; i <= Integer.MAX_VALUE; i++)
            sum += i;
        return sum;
    }
    ```
    작동하기는 하나 매우 느리다. sum 변수를 long이 아닌 Long으로 선언해서, sum에 i를 더할때마다 (오토박싱이 발생해) Long 인스턴스를 생성해내기 때문이다.
