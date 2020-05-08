# 프로그래머스 '카드 게임' 풀이

* 문제 출처 : [프로그래머스 Lv4.카드 게임](https://programmers.co.kr/learn/courses/30/lessons/42896)

```java
class Solution {
    public int solution(int[] left, int[] right) {
        int answer = 0;
        int[][] dp = new int[left.length + 1][right.length + 1];

        for(int i = 1; i < left.length + 1; i++) {
            for(int j = 1; j < right.length + 1; j++) {
                if(left[left.length - i] > right[right.length - j]) {
                    dp[i][j] = Math.max(dp[i][j - 1] + right[right.length - j], Math.max(dp[i - 1][j], dp[i - 1][j - 1]));
                }else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - 1]);
                }
            }
        }
        answer = dp[left.length][right.length];
        return answer;
    }
}
```

    
              