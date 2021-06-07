# 이펙티브자바 3장 - 모든 객체의 공통 메서드

## 아이템10 equals는 일반 규약을 지켜 재정의하라

* `equals()`는 재정의하기 쉬워 보이지만 자칫하면 끔찍한 결과를 초래한다. 다음 중 하나에 해당한다면 재정의하지 않는 것이 최선이다
    + 각 인스턴스가 본질적으로 고유하다 ex) `Thread`
    + 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다. ex) java.util.regex.Pattern
    + 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다. ex) List / AbstractList
    + 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다. 이런 상황에서 equals가 행여라도 호출하는 걸 방지하고 싶다면 아래와 같이 구현하자.
        ```java
        @Override public boolean equals(Object o) {
            throw new AssertionError();
        }
        ```

* equals를 재정의가 필요한 경우는, **두 객체가 물리적으로 같은가가 아니라 논리적으로 같은가를 확인해야 하는데, 상위 클래스의 equals에서 재정의되지 않았을 때**이다. ex) `Integer`, `String`같은 값 클래스
    + 프로그래머는 값 클래스의 객체가 같은지를 알고 싶은게 아니라 값이 같은지를 알고 싶어할 것.
    + 논리적 동치성을 확인하도록 재정의해두면, 값을 비교하길 원하는 프로그래머의 기대에 부응할 뿐만 아니라 `Map`의 키와 `Set`의 원소로 사용할 수 있게 된다.
    + 값 클래스라고 해도, 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장한다면 재정의할 필요가 없다.(물리적으로 같다면 어차피 논리적으로도 같을 것이기 때문에) ex) Enum 

* equals를 재정의할 때 따라야 할 규약 : 반사성(reflexivity), 대칭성(symmetry), 추이성(transitivity), 일관성(consistency), null-아님
    + 반사성(reflexivity)
        * *null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.*
        * 객체는 자기 자신과 같아야 한다는 뜻이다. 일부러 어기는 것이 더 힘들 것 같다.
        * 이 요건을 어긴 클래스의 인스턴스를 컬렉션에 넣은 다음 `contains()`를 호출하면 false를 반환할 것.
    + 대칭성(symmetry)
        * *null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.*
        ```java
        public final class CaseInsensitiveString {
            private final String s;

            public CaseInsensitiveString(String s){
                this.s = Objects.requireNonNull(s);
            }

            @Override public boolean equals(Object o) {
                if (o instanceof CaseInsensitiveString)
                    return s.equalsIgnoreCase( ((CaseInsensitiveString)o).s );
                if (o instanceof String)
                    return s.equalsIgnoreCase((String) o);
                return false;
            }
        }
        ```
        * 위와 같이 `CaseInsensitiveString` 클래스가 있다고 하자. 이 클래스는 일반 문자열과도 비교를 시도한다. 그런데 아래처럼 `CaseInsensitiveString`과 `String`의 인스턴스가 있다면?

        ```java
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String s = "polish";
        ```        
        * 이 경우 `cis.equals(s)`는 true인 반면 `s.equals(cis)`는 false이므로 대칭성을 위배한다.   
        cis를 컬렉션에 넣는다면 어떻게 될까?

        ```java
        List<CaseInsensitiveString> list = new ArrayList<>();
        list.add(cis);
        ```

        * `list.contains(s)`를 호출하면 (저자의 말에 따르면) OpenJDK에서는 false를 반환한다고 한다. 그러나 이는 순전히 어떻게 구현하느냐에 따른 차이이기 때문에, OpenJDK 버전이 바뀌거나 다른 JDK에서는 true를 반환하거나 예외를 던질 수도 있다. *equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할 지 알 수 없다.*

        * 그렇다면 이 문제를 해결하려면 어떻게 해야 할까? `String`과 연동하겠다는 허황된 꿈을 버려야 한다.
        ```java
        @Override public boolean equals(Object o) {
            return o instanceof CaseInsensitiveString && 
                ((CaseInsensitiveString)o).s.equalsIgnoreCase(s);
        }
        ```

        
    + 추이성(transitivity)
        * *null이 아닌 모든 참조 값 x,y,z에 대해, x.equals(y)가 true이고 y.equals(z)가 true면 x.equals(z)도 true다.*
        * 상위 클래스에는 없는 새로운 필드를 하위 클래스에 추가하는 상황을 생각해보자. `Point` 클래스와, color 필드를 추가한 `ColorPoint` 클래스는 아래와 같다.
        ```java 
        public class Point {
            private final int x;
            private final int y;

            public Point(int x, int y) {
                this.x = x;
                this.y = y;
            }

            @Override public boolean equals(Object o) {
                if(!(o instanceof Point))
                    return false;
                Point p = (Point)o;
                return p.x == x && p.y == y;
            }

            // ... 나머지 코드 생략
        }
        ```

        ```java
        public class ColorPoint extends Point {
            private final Color color;

            public ColorPoint(int x, int y, Color color) {
                super(x,y);
                this.color = color;
            }

            // ... 나머지 코드 생략
        }
        ```

        * 이 상황에서 equals는 어떻게 해야 할까? 그대로 둔다면 Point의 equals를 상속받아 색상 정보를 무시한 채 비교를 수행한다. equals 규약을 어긴 것은 아니지만, 중요한 정보를 놓치고 비교를 하는 상황. 다음 코드처럼 equals를 작성할 경우를 생각해보자.

        ```java
        @Override public boolean equals(Object o) {
            if (!(o instanceof ColorPoint))
                return false;
            return super.equals(o) && ((ColorPoint)o).color == color;
        }
        ```
        ```java
        Point p = new Point(1,2);
        ColorPoint = new ColorPoint(1,2,Color.RED);
        ```
        
        * 이 경우 대칭성을 위배하게 된다. 그렇다면 `ColorPoint.equals()`가 Point와 비교할 때는 색상을 무시하도록 하면 해결될까?

        ```java
        @Override public boolean equals(Object o) {
            if(!(o instanceof Point))
                return false;

            // o가 ColorPoint가 아닌 Point이면 색상을 무시하고 비교한다.    
            if(!(o instanceof ColorPoint))
                return o.equals(this);
            // o가 ColorPoint면 색상까지 비교한다.
            return super.equals(o) && ((ColorPoint) o).color == color;
        }
        ```

        ```java
        ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
        Point p2 = new Point(1,2);
        ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
        ```

        * 이 경우 대칭성은 지켜주지만, 추이성을 깨버린다. *p1.equals(p2) == true, p2.equals(p3) == true이나, p1.equals(p3) == false이기 때문이다.*
        * 뿐만 아니라, `Point`의 또 다른 하위 클래스 `SmellPoint`를 같은 방식으로 구현한 다음, `cp.equals(sp)`를 호출하면 무한 재귀에 빠져 `StackOverflowError`를 일으킨다.
        * 근본적인 해결책은 무엇일까? **구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.** 혹자는 이 말을 자칫 equals 안의 `instanceof`를 `getClass()`로 바꾸면 규약을 지키고 값을 추가하면서 구체 클래스를 상속할 수 있다고 오해할 수 있다.

        ```java
        @Override public boolean equals(Object o) {
            if(o == null || o.getClass() != getClass())
                return false;
            Point p = (Point) o;
            return p.x == x && p.y == y;
        }
        ```
        * 괜찮아 보이나 실제로 활용할 수는 없다. Point의 하위 클래스는 정의상 여전히 Point이므로 여디서든 Point로써 활용될 수 있어야 한다. 주어진 점이 (반지름이 1인) 단위 원 안에 있는지를 판별하는 메서드를 구현했다고 가정해보자.

        ```java
        public static final Set<Point> unitCircle = Set.of(
            new Point(1, 0), new Point(0, 1), new Point(-1,0), new Point(0,-1);
        )

        public static boolean onUnitCircle(Point p){
            return unitCircle.contains(p);
        }
        ```

        * 이제 여기서 Point를 확장하면,

        ```java
        public class CounterPoint extends Point {
            private static final AtomicInteger counter = new AtomicInteger();

            public CounterPoint(int x, int y) {
                super(x,y);
                counter.incrementAndGet(); // 여기서 이걸 왜 세는건지 모르겠으나 ..
            }
            public static int numberCreated() { return counter.get(); }
        }
        ```

        * *어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다. 따라서 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야 한다.(리스코프 치환 원칙)* 그런데 CounterPoint의 인스턴스를 `onUnitCircle()`에 넘기면 false를 반환한다. `Set`을 포함한 대부분의 컬렉션은 이 작업에 equals를 사용하기 때문이다. `instanceof`으로 구현했다면 제대로 동작했을 것이다.

        * 구체 클래스의 하위 클래스에서 값을 추가할 방법은 없지만 괜찮은 우회 방법이 있다. 컴포지션을 사용하는 것. `Point`를 상속하는 대신 `Point`를 private field로 두고, `ColorPoint`와 같은 위치의(같은 x,y를 가진) 읿반 `Point`를 반환하는 뷰(view) 메서드를 public으로 추가하는 식이다.

        ```java
        public class ColorPoint {
            private final Point point;
            private final Color color;

            public ColorPoint(int x, int y, Color color) {
                point = new Point(x,y);
                this.color = Objects.requireNonNull(color);
            }

            /*
            * 이 ColorPoint의 Point 뷰를 반환한다.
            */
            public Point asPoint() {
                return point;
            }

            @Override public boolean equals(Object o) {
                if(!(o instanceof ColorPoint))
                    return false;
                ColorPoint cp = (ColorPoint) o;
                return cp.point.equals(point) && cp.color.equals(color); // asPoint() 써야하는것 아닌가..?
            }
            // 나머지 코드는 생략 
        }
        ```

        * 자바 라이브러리에도 구체 클래스를 확장해 값을 추가한 클래스가 종종 있다. 예컨대, `java.sql.Timestamp`는 `java.util.Date`를 확장한 후 nanoseconds 필드를 추가헀다. 그 결과, Timestamp의 equals는 대칭성을 위배하며, `Date` 객체와 한 컬렉션에 넣거나 서로 섞어 사용하면 엉뚱하게 동작할 수 있다. (이렇게 설계한 것은 실수니, 따라하지 말자.)

            *추상 클래스의 하위 클래스에서라면 equals 규약을 지키면서도 값을 추가할 수 있다. 상위 클래스를 직접 인스턴스로 만드는게 불가능하다면 지금까지 이야기한 문제들은 일어나지 않는다.*

    + 일관성(consistency)

        * *null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.*

        * 두 객체가 같다면 수정이 이뤄지지 않는 한 영원히 같아야 한다는 뜻. 
        * equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다. 예를 들어, `java.net.URL`의 equals는 주어진 URL과 매핑된 호스트의 IP를 이용해 비교한다. 호스트 이름을 IP주소로 바꾸려면 네트워크를 통해야 하는데, 그 결과가 항상 같다고 보장할 수 없다. 이 역시 설계 오류이다.
    + null-아님
        * *null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.*
        * equals에서 null을 체크하는 명시적 방법보다, `instanceof`를 활용하는 묵시적 방법이 낫다.

        ```java
        @Override public boolean equals(Object o) {
            if( o== null)
                return false; //명시적 null 검사 - 필요 없다!
        }
        ```

        ```java
        @Override public boolean equals(Object o) {
            if(!(o instanceof MyType))
                return false;
            MyType mt = (MyType) o;
            // .. 나머지 생략
        }
        ```

        `instanceof`는 (두번째 피연산자와 무관하게) 첫번째 피연산자가 null이면 false를 반환하기 때문.


