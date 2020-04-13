# 프로그래머스 '네트워크' 풀이

```java
import java.util.*;

class Solution {
    public int solution(int n, int[][] computers) {
        Graph graph = new Graph(n);
        graph.connectEdges(computers); 
        return graph.bfs();        
    }
}

class Graph {
    private int numberOfNodes;
    private LinkedList[] adjList;
    private boolean[] visited;
    
    public Graph(int numberOfNodes){
        this.numberOfNodes = numberOfNodes;
        this.adjList = new LinkedList[numberOfNodes];
        this.visited = new boolean[numberOfNodes];
        
        for(int i = 0; i < numberOfNodes; i++){
            adjList[i] = new LinkedList<Integer>();
            visited[i] = false;
        }
    }
    
    public void connectEdges(int[][] arr){
        for(int i = 0; i < arr.length; i++){
            for(int j = 0; j < arr.length; j++){
                if( i != j && arr[i][j] != 0 )
                    adjList[i].add(j);
            }
        }
    }
    
    public int bfs() {
        int requiredNetworks = 0;
        
        Queue<Integer> q = new LinkedList<Integer>();
        for(int i = 0; i < numberOfNodes; i++){
            
            if(!visited[i]){
                requiredNetworks++;
                visited[i] = true;
                q.add(i);
            }
            
            while(!q.isEmpty()) {
                int temp = q.poll();
                for(int j = 0; j < adjList[temp].size(); j++){
                    Integer k = (Integer)adjList[temp].get(j);
                    if(!visited[k]) {
                        visited[k] = true;
                        q.add(k);
                    }
                }
            }
            if(isAllNodesVisited()) return requiredNetworks;
        }
        return 0;
    }
    
    private boolean isAllNodesVisited(){
        for(int i = 0; i < numberOfNodes; i++){
            if(!visited[i]) return false;
        }
        return true;
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