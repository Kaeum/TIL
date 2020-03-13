# 프로그래머스 '디스크 컨트롤러' 풀이

```java
import java.util.*;

class Solution {
    public int solution(int[][] jobs) {
        int answer=0;
        int time = 0;
        List<Job> jobList = new ArrayList<>();
        for(int i = 0; i < jobs.length; i++){
            jobList.add(new Job(jobs[i][0], jobs[i][1]));
        }
        Collections.sort(jobList);
        
        while(jobList.size() > 0){
            for(int i = 0; i < jobList.size(); i++){
                if(jobList.get(i).reqTime <= time){
                    time += jobList.get(i).prcTime;
                    answer += time - jobList.get(i).reqTime;
                    jobList.remove(i);
                    break;
                }
                if(i==jobList.size()-1){
                    time++;
                }
            }
        }
        
        return answer / jobs.length;
    }
}

class Job implements Comparable<Job> {
    int reqTime, prcTime;
        
    public Job(int reqTime, int prcTime){
        this.reqTime = reqTime;
        this.prcTime = prcTime;
    }
    
    @Override
    public int compareTo(Job job){
        if(this.prcTime - job.prcTime == 0){
            return this.reqTime - job.reqTime;
        } else{
            return this.prcTime - job.prcTime;
        }
    }
}
```
***
# Today-I-Learned
* 정렬하고자 하는 클래스에 `Comparable` 인터페이스를 구현하고 `ArrayList`에 , `Collectins.sort`를 사용하였다.
  * 처음에는 `PriortyQueue`를 사용하려했으나, Queue안의 모든 요소들을 (요소를 제거하지 않고) 순회하려고 하니 어려움이 있었다.
  * 아래 반복문에서 보듯이, if문에 진입할 경우에만 해당 요소를 제거해야하는데, `poll()`을 쓰지 않고 `peek()`을 통해서 모든 요소를 순회하기는 쉽지 않다고 생각했다.
  ```java
    while(jobList.size() > 0){
      for(int i = 0; i < jobList.size(); i++){
          if(jobList.get(i).reqTime <= time){
              time += jobList.get(i).prcTime;
              answer += time - jobList.get(i).reqTime;
              jobList.remove(i);
              break;
          }
          if(i==jobList.size()-1){
              time++;
          }
      }
    }
    ```
    * `Queue`에도 `peekFirst()`, `peekLast()` 등의 메서드가 존재하나 (특정 요소를 poll/remove하지 않고) 모든 요소를 순회하는데에는 어려움이 있으며, `ArrayList`에 넣을 경우 index를 활용할 수 있어 훨씬 편리하였다.
