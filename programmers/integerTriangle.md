# 프로그래머스 '정수 삼각형' 풀이

* 문제 출처 : [프로그래머스 Lv3.정수 삼각형](https://programmers.co.kr/learn/courses/30/lessons/43105)

```java
class Solution {
    int[][] memo;
    
    public int solution(int[][] triangle) {
        int answer = 0;
        memo = new int[triangle.length][triangle.length];
        
        memo[0][0] = triangle[0][0];
        for(int i = 1; i < memo.length; i++){
            memo[i][0] = memo[i-1][0]  + triangle[i][0];
            memo[i][i] = memo[i-1][i-1] + triangle[i][i];
        }
        
        for(int i = 2; i < memo.length; i++){
            for(int j = 1; j < i; j++){
                memo[i][j] = Math.max(memo[i-1][j-1], memo[i-1][j]) + triangle[i][j];
            }
        }
        
        for(int i = 0; i < memo.length; i++){
            if(memo[memo.length - 1][i] > answer) answer = memo[memo.length - 1][i]; 
        }
        
    return answer;
    }
}
```
***
# Today-I-Learned
* bottom-up 방식의 dp 문제였다. 삼각형 양 변을 먼저 초기화한 후 memoization을 사용해 풀었다. 

    
              