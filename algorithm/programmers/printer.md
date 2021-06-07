# 프로그래머스 '프린터' 풀이

```java
import java.util.List;
import java.util.ArrayList;

class Solution {
    public int solution(int[] priorities, int location) {
        int answer = 0;

        List<Docs> list = new ArrayList<>();

        for(int i = 0; i < priorities.length; i++){
            list.add(new Docs(i, priorities[i]));
        }

        Docs temp = new Docs();

        for(int i = 0; i < list.size(); i++){
            for(int j = i+1; j < list.size(); j++){
                if(list.get(i).getPriority() < list.get(j).getPriority()){
                    temp = list.get(i);
                    list.remove(i);
                    list.add(i, new Docs(999,0));
                    list.add(temp);
                    break;
                }
            }
        }

        list.removeIf(x -> x.getPriority() == 0);

        for(int i = 0; i < list.size(); i++){
            if(location == list.get(i).getId()){
                answer = i + 1;
            }
        }
        return answer;
    }
}

class Docs {
    int id;
    int priority;
    public Docs(){

    }

    public Docs(int id, int priority){
        this.id = id;
        this.priority = priority;
    }

    public int getId(){
        return this.id;    
    } 
    public int getPriority(){
        return this.priority;    
    } 
}
```
***
# Today-I-Learned
* 정말 어거지로 풀었던 문제. 큐를 사용하지 않고 리스트로 풀다보니 지저분한(?) 로직이 많이 들어갔다.
  * 우선순위를 비교한 후, 더 높은 우선순위의 문서가 있으면 제일 뒤로 보내는 로직
  * 문서를 제일 뒤로 보내는 로직을 구현할 때 하기의 순서대로 구현(누가봐도 어거지)
    1) 해당 인덱스의 문서를 temp 변수에 담아둔 후 리스트에서 제거 
    2) 제거한 위치에 우선순위(priority)가 0인 문서를 강제 삽입
    3) 가장 뒤에 temp를 삽입
  * for문을 전부 빠져나온 후, 우선순위가 0인 문서들을 전부 리스트에서 제거.
* 다른 풀이들을 살펴보니, 대부분 큐를 활용하여 해결하였고 location의 값을 바꿔가며 답을 찾아내는 과정이 많았음
* location을 바꾸는 접근은 생각도 못했다.. 아직 멀었다고 느낌. 머리에서 풀이가 잊혀질 쯤 다시 풀어볼 것.
