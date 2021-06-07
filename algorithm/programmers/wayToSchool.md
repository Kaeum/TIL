# 프로그래머스 '등굣길' 풀이

* 문제 출처 : [프로그래머스 Lv3.등굣길](https://programmers.co.kr/learn/courses/30/lessons/42898)

```java
class Solution {
    public int solution(int m, int n, int[][] puddles) {
        int[][] memo = new int[n][m];
        
        memo[0][0] = 1;
        for(int[] puddle : puddles){
            memo[puddle[1]-1][puddle[0]-1] = -1;
        }
        
        for(int i = 0; i < n; i++){
            for(int j = 0; j < m; j++){
                if(memo[i][j] != -1){
                    if(i>0 && memo[i-1][j] != -1) memo[i][j] += memo[i-1][j] % 1000000007;
                    if(j>0 && memo[i][j-1] != -1) memo[i][j] += memo[i][j-1] % 1000000007;
                }
            }
        }
        
        return memo[n-1][m-1] % 1000000007;
    }
}
```
***
# Today-I-Learned
* puddle인 지역은 -1로 표시하고, 위쪽(↑)과 왼쪽(←)의 가능한 길의 수를 합치는 방식으로 풀었다.   
* 2중 for-loop에서 1,000,000,007로 나누는 것이 형변환(casting)이 일어나지 않게 하기 위함이라는 것은 이해가 되나, 마지막 return 문에도 1,000,000,007로 나눠줘야 효율성 테스트를 통과하는 이유는 아직 잘 모르겠다.

    
              