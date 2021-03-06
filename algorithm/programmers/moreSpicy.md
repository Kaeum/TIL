# 프로그래머스 '더 맵게' 풀이

```java
import java.util.*;

class Solution {
    public int solution(int[] scoville, int K) {
        int answer = 0;
        
        Queue<Integer> que = new PriorityQueue<Integer>(scoville.length);
        for(int i = 0; i < scoville.length; i++){
            que.add(scoville[i]);
        }
        
        while(!que.isEmpty()){
            if(que.peek() < K){
                que.add(que.poll() + que.poll() * 2);
                if(que.size() == 1 && que.peek() < K){
                    answer = -1;
                    break;
                }
                answer++;
                continue;
            }
            
            break;
        }
        
        
        return answer;
    }
}
```
***
# Today-I-Learned
* if(que.peek() < K) 문과 break는 while문의 조건 안으로 집어넣을 경우 아래와 같이 코드가 더 깔끔해진다.
```java
        while(!que.isEmpty() && que.peek() < K){
                que.add(que.poll() + que.poll() * 2);
                if(que.size() == 1 && que.peek() < K){
                    answer = -1;
                    break;
                }
                answer++;
        }
```
* 무작정 for문을 중복해서 풀기보다, 적합한 자료구조를 이용할 경우 훨씬 쉽게 접근할 수 있다.
