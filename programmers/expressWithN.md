# 프로그래머스 '네트워크' 풀이

* 문제 출처 : [프로그래머스 Lv3.N으로 표현](https://programmers.co.kr/learn/courses/30/lessons/42895)

```java
import java.util.*;

class Solution {
    Map<Integer, ArrayList<Integer>> hashMap = new HashMap<>();
    
    public int solution(int N, int number) {
        int answer = -1;
        for(int i = 1; i < 9; i++){
            hashMap.put(i, new ArrayList<Integer>());
        }
        
        hashMap.get(1).add(N);
        for(int i = 2; i < 9; i++){
            for(int j = 0; j < i; j++){
                if(j==0) {
                    hashMap.get(i).add( Integer.parseInt( hashMap.get(i-1).get(0)+""+N) );
                    continue;
                }
                for(Integer num1 : hashMap.get(j)) {
                    for(Integer num2 : hashMap.get(i - j)){
                        List<Integer> list = hashMap.get(i);
                        list.add(num1 + num2);
                        list.add(num1 - num2);
                        list.add(num1 * num2);
                        if(num2 != 0) list.add(num1 / num2);
                    }
                }
            }
        }
        
        for(int i = 1; i < 9; i++){
            if(hashMap.get(i).contains(number)) return i;
        }
        
        return answer;
        }
}
```
***
# Today-I-Learned
* for-loop를 4번이나 중첩하면서 푸는 방식은 꽤나 무식(?)하다고 생각하는데, 아직 더 나은 풀이를 생각해내지 못했다. 효율성 테스트가 있었다면 통과하지 못할 수도 있었을 것 같다.
* 처음에는 이 문제가 어떻게 DP(Dynamic Programming) 문제인지 잘 감이 안왔으나, subproblem을 아래와 같이 나누었다.   

    -DP(n)을 N을 n개 사용해서 만들 수 있는 경우의 수의 집합(Set)이라고 하자.   
    -A◎B를 Set A와 Set B를 사칙연산하여 만들 수 있는 경우의 수라고 하자.   

    그러면, DP(5) = NNNNN(N을 5번 붙여 만든 수) + DP(1)◎DP(4) + DP(2)◎DP(3) + DP(3)◎DP(2) + DP(4)(1) 이 된다.   
    마찬가지로 DP(4) = NNNN + DP(1)◎DP(3) + DP(2)◎DP(2) + DP(3)◎DP(1) 등등이 성립한다.   
    다만, 사칙 연산의 결과 Set을 저장한 후, 상위문제에서 그 결과값을 가져와서 다시 사칙연산을 하기 때문에, for-loop가 굉장히 많이 필요했으며, 수행시간도 꽤 길었다.

* ArrayList 대신 HashSet 등을 사용했다면 더 좋았을 것 같다.    

    
              