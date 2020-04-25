# 프로그래머스 '장식 타일물' 풀이

* 문제 출처 : [프로그래머스 Lv3.장식 타일물](https://programmers.co.kr/learn/courses/30/lessons/43104)

```java
class Solution {
    long[] memory;
    
    public long solution(int N) {
        initMemoryWithSize(N);
        getLengthOfAllTiles();

        return 2 * (memory[N] + memory[N-1]);
    }
    
    private void initMemoryWithSize(int n){
        memory = new long[n+1];
    }
    
    private void getLengthOfAllTiles(){
        for(int i = 0; i < memory.length; i++){
            if(i < 2){
                memory[i] = 1;
            } else {
                memory[i] = memory[i-1] + memory[i-2];    
            }
        }
    }
}

```
***
# Today-I-Learned
* 문제가 길지만 그냥 피보나치 수열 문제이다. Bottom-up 방식을 이용해 풀었다.

    
              