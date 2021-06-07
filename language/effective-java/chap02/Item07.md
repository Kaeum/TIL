# 이펙티브자바 2장 - 객체 생성과 파괴

## 아이템7 다 쓴 객체 참조를 해제하라

* 아래 코드는 어디가 문제일까?
    ```java
    public class Stack {
        private Object[] elements;
        private int size = 0;
        private static final int DEFAULT_INITIAL_CAPACITY = 16;

        public Stack() {
            elements = new Object[DEFAULT_INITIAL_CAPACITY];
        }

        public void push(Object e) {
            ensureCapacity();
            elements[size++] = e;
        }

        public Object pop() {
            if(size == 0) throw new EmptyStackException();
            return elements[--size];
        }
        /*
        * 원소를 위한 공간을 적어도 하나 이상 확보한다.
        * 배열 크기를 늘려야 할 때마다 대략 두배씩 늘린다.
        */
        private void ensureCapacity() {
            if(elements.length == size)
                elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
    ```
    메모리 누수가 일어나고 있는 코드이다. 스택이 늘어났다가 줄어들 때, 꺼내진 객체들을 GC가 회수하지 않는다. (다 쓴 참조를 스택이 여전히 가지고 있기 때문) 객체 참조가 한개가 살아있으면 그 객체만 회수하지 못하는 것이 아니다. 그 객체가 참조하는 모든 객체(그리고 그 모든객체들이 참조하는 모든 객체)를 회수하지 못한다.
    제대로 구현하려면 `pop()`을 아래와 같이 바꿔주어야 한다.
    ```java
        public Object pop() {
            if(size == 0)
                throw new EmptyStackException();
            Object result = elements[--size];
            elements[size] = null; //다 쓴 참조 해제.
            return result;                
        }
    ```
    그러나 위와 같이 프로그래머가 객체 참조를 null 처리하는 일은 예외적인 경우여야 한다. (앞의 예에서는 스택이 자기 메모리를 직접 관리하기 때문! *메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다.) 가장 좋은 방법은 유효 범위(scope) 밖으로 밀어내는 것.*

* 캐시 역시 메모리 누수를 일으키는 주범
    + 객체 참조를 캐시에 넣어 놓은 사실을 까먹고, 한참동안 다 쓴 객체를 놔두는 경우.
    + 해법은 여러가지 : `WeakHashMap`, `ScheduledThreadPoolExecutor`, `LinkedHashMap`의 `removeEldestEntry`, `java.lang.ref` ...
    + 무슨 말인지 이해가 잘 안간다. 그냥 서드파티 캐시를 쓰자..
    + `WeakHashMap` 과 관련하여 참조할만한 글 : [Java reference와 GC](https://d2.naver.com/helloworld/329631)

* 세번째 주범은 리스너 or 콜백
    + 클라이언트가 콜백을 등록만 하고 명확히 해주지 않으면, 콜백은 계속 쌓여갈 것
    + 이 때도 콜백을 약한 참조(weak refenrece)로 지정하면 GC가 수거해갈 수 있다.(WeakHashMap에 키로 저장)
