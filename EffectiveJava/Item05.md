# 이펙티브자바 2장 - 객체 생성과 파괴

## 아이템5 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

* 하나 이상의 자원(아래 예에서 dictionary)에 의존할 때, 아래와 같이 구현한다면?

    + 정적 유틸리티 클래스(아이템4)
    ```java
    public class SpellChecker {
        private static final Lexicon dictionary = ...;

        private SpellChecker() {} // 객체 생성 방지

        public static boolean isValid(String word) { ... }
        public static List<String> suggestions(String typo) { ... }
    }
    ```
    + 싱글턴(아이템3)
    ```java
    public class SpellChecker {
        private final Lexicon dictionary = ...;

        private SpellChecker(...) {}
        public static SpellChecker INSTANCE = new SpellChecker(...);
        
        public static boolean isValid(String word) { ... }
        public static List<String> suggestions(String typo) { ... }        
    }
    ```

    *사전을 하나만 쓴다고 가정한 설계. 사전을 교체할 경우 어떻게 대응할 것인가?*

* 클래스가 여러 자원 인스턴스를 지원하고, 클라이언트가 원하는 자원을 사용해야 한다면? *의존 객체 주입*이 필요하다.
    + 생성자를 활용한 의존 객체 주입
    ```java
    public class SpellChecker {
        private final Leexicon dictionary;

        public SpellChecker(Lexicon dictionary) {
            this.dictionary = Objects.requireNonNull(dictionary);
        }

        public static boolean isValid(String word) { ... }
        public static List<String> suggestions(String typo) { ... }    
    }
    ```    

* 의존 객체 주입을 활용하면 불변(아이템17)을 보장받을 수 있으며, 클라이언트가 안심하고 의존 객체들을 공유할 수 있다.

* 정적 팩터리(아이템 1), 빌더(아이템 2), 팩터리 메서드(Supplier<T> 인터페이스 등)을 응용할 수도 있다.
* 스프링, 주스, 대거 같은 의존성 주입(Dependency Injection)을 지원하는 프레임워크를 사용하는 것도 좋은 방법이다.
