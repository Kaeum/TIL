# 프로그래머스 '네트워크' 풀이

* 문제 출처 : [프로그래머스 Lv3.네트워크](https://programmers.co.kr/learn/courses/30/lessons/43162)

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
* 비연결 그래프를 전부 방문하는 데 몇 개의 네트워크(for-loop)가 필요한 지 구하는 방식으로 접근하였다.
* 전형적인 연결 성분 개수를 구하는 문제로, DFS로도 풀이가 가능하다.