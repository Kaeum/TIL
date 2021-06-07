# 프로그래머스 '주식 가격' 풀이

* 문제 출처 : [프로그래머스 Lv2.주식 가격](https://programmers.co.kr/learn/courses/30/lessons/42584)

```java
class Solution {
    public int[] solution(int[] prices) {
        int[] answer = new int[prices.length]; 
        
        for(int i = 0; i < prices.length; i++){
            if(i == prices.length - 1){
                answer[i] = 0;
                break;
            }
            for(int j = i+1; j < prices.length; j++){
                if(j == prices.length - 1){
                    answer[i] = prices.length - 1 - i;
                    break;
                }
                
                if(prices[i] > prices[j]){
                    answer[i] = j - i;
                    break;
                }
            }
        }
    return answer; 
    }
}
```

    
              