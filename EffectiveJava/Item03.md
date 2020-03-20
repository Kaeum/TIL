# 이펙티브자바 2장 - 객체 생성과 파괴

## 아이템3 private 생성자나 열거 타입으로 싱글턴임을 보증하라

* 싱글턴이란, 인스턴스를 오직 하나만 생성할 수 있는 클래스   
    + logging, authentication과 같이 유일해야 하는 시스템 컴포넌트가 전형적인 예시
    + 가짜(mock) 객체를 만들기 힘드므로 클라이언트 테스트가 힘들다(?)

* 싱글턴을 구현하는 방식
    + public static final 필드 방식의 싱글턴
    ```java
    public class Elvis{
        public static final Elvis INSTANCE = new Elvis();
        private Elvis() { ... }

        public void doSomething(){ ... }
    }    
    ```

    + 정적 패터리 방식의 싱글턴
    ```java
    public class Elvis {
        private static final Elvis INSTANCE = new Elvis();
        private Elvis() { ... }
        public static Elvis getInstance() { return INSTANCE; }

        public void doSomething(){ ... }
    }
    ```

    *위의 두 방식은 reflection을 통해 private 생성자를 호출할 경우 싱글턴을 보장하지 못한다.   
    또한 직렬화/역직렬화하는 과정에서 가짜 인스턴스가 만들어질 수 있는데, 이를 방지하기 위해서는 `readResolve` 메서드를 제공하고 모든 인스턴스 변수를 transient라고 선언해야한다.*

    + Enum을 사용하는 싱글턴
    ```java
    public enum Elvis {
        INSTANCE;

        public void doSomething(){ ... }
    }
    ```
    *이 책에서 가장 바람직하다고 말하는 방식. 직렬화/역직렬화 상황이나 reflection 공격에서도 문제가 없다.*

* 책에는 소개되어있지 않지만, 싱글턴을 구현하는 (훌륭한) 방식이 더 있다.
    + DCL(Double-Checked-Locking) 싱글턴

    ```java
    public class Singleton {
        private static Singleton INSTANCE;
        private Singleton(){}

        public static Singleton getInstance() {
            if(INSTANCE == null) {
                synchronized (Singleton.class) {
                    if(INSTANCE == null){
                        INSTANCE = new Singleton();
                    }
                }
            }
            return INSTANCE;
        }
    }
    ```
    *`getInstance()` 메서드 자체를 synchronized로 선언할 경우의 성능 문제를 개선하기 위한 패턴. 인스턴스가 생성되지 않은(null) 경우에만 lock을 잡고 인스턴스를 생성한다. 그러나 이 방식은 멀티스레드 환경에서 문제가 될 수 있다. 스레드마다 별도 메모리 공간(Cache)에서 변수를 읽어오기 때문에 변수의 값이 달라지면서 문제가 생기는 것.*
    
    + DCL with volatile (JDK 1.5 이상)
    ```java
        public class Singleton {
        private volatile static Singleton INSTANCE;
        private Singleton(){}

        public static Singleton getInstance() {
            if(INSTANCE == null) {
                synchronized (Singleton.class) {
                    if(INSTANCE == null){
                        INSTANCE = new Singleton();
                    }
                }
            }
            return INSTANCE;
        }
    }
    ```
    *이 경우 `volatile` 변수(INSTANCE)를 Cache에서 변수를 참조하지 않고 메인 메모리에서 변수를 참조한다. 따라서 멀티스레드 환경에서도 문제가 되지 않는다.(아직까지는)*

    + LazyHolder Singleton
    ```java
    public class Singleton(){
        private Singleton(){}

        public static Singleton getInstance(){
            return LazyHolder.INSTANCE;
        }
        
        private static class LazyHolder{
            private static final INSTANCE = new Singleton();
        }
    }
    ```
    *가장 완벽하다고 평가받는 방법. LazyHolder 클래스(내부 클래스)는 외부의 클래스(Singleton 클래스)가 로딩될 때 초기화되지 않고, 클래스의 static 변수는 클래스를 로딩할 때 초기화되는 것을 이용한 기법이다. 즉, `getInstance()`가 호출되면 LazyHolder 클래스가 로딩되고, 이 때 static 변수인 INSTANCE가 초기화된다. 클래스를 로딩하는건 JVM의 영역이므로 Thread-safe하다.*
