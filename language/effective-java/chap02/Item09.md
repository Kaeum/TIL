# 이펙티브자바 2장 - 객체 생성과 파괴

## 아이템9 try-finally보다는 try-with-resources를 사용하라

* `close()`를 호출해 직접 닫아줘야 하는 자원들(`InputStream`, `OutputStream`, `java.sql.Connection` 등)이 제대로 닫힘(closed)을 보장하는 수단으로 try-finally가 쓰였다.

    ```java
    // Code 9-1
    static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try{
            return br.readLine();
        } finally {
            br.close();
        }
    }
    ```

    *나쁘지 않지만, 자원을 하나 더 사용한다면?*

    ```java
    // Code 9-2
    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try{
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while (( n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
    ```

    + 위의 두 코드(9-1,9-2)는 결점이 있는 코드다. **왜냐하면 예외는 try문과 finally문 모두에서 발생할 수 있기 때문이다.** 예컨대 9-1에서 기기에 물리적인 문제가 생긴다면 `br.readLine()`과 `br.close()` 모두에서 예외가 발생할 수 있다. 이런 경우 두 번째 예외가 첫 번째 예외를 삼켜버려서, 첫번째 예외에 대한 정보를 추적할 수 없다.(디버깅이 매우 힘들어진다.)   

* 대안은 try-with-resources + `AutoCloseable` 인터페이스를 구현하는 것.

    ```java
    // Code 9-3 : Modified Code 9-1 with try-with-resources
    static String firstLineOfFile(String path) throws IOException {
        try ( BufferedReader br = new BufferedReader( new FileReader(path) ) ){
            return br.readLine();
        }
    }
    ```

    
    ```java
    // Code 9-4 : Modified Code 9-2 with try-with-resources
    static void copy(String src, String dst) throws IOException {
        try (InputStream in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)){
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        }
    }
    ```
    
    + try-with-resources를 사용하면 가독성이 뛰어날 뿐만 아니라, 디버깅에도 더 유리하다. 위에서 예를 들었던 예외 발생 케이스의 경우, 즉 `firstLineOfFile()`에서 `readLine()`과 `close()` 양쪽에서 모두 예외가 발생한다면 `close()`에서 발생한 예외는 숨겨지고 `readLine()`에서 발생한 예외가 기록된다. 이 때, 숨겨진 예외들은 버려지지 않고 스택 추적 내역에 '숨겨졌다(suppressed)'는 꼬리표를 달고 출력된다. 자바 7에서 `Throwable`에 추가된 `getSuppressed()`를 이용하여 가져올 수도 있다.

**결론 : Java 1.7부터는 예외없이 try-with-resources를 사용하자. 가독성이 높아지며, 예외 정보도 유용하며, 자원 회수도 정확하다.**    