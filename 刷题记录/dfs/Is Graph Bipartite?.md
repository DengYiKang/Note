# Is Graph Bipartite?

### 问题

判断一个图是否为二分图。

### 解决方案：dfs，时间复杂度$O(n)$

dfs进行染色，染色数组`int[] type`即可以表示当前的染色状态又可以表示是否被dfs过。

```java
class Solution {
    public boolean isBipartite(int[][] graph) {
        int N=graph.length;
        int[] type=new int[N];
        Arrays.fill(type, -1);
        for(int i=0; i<N; i++){
            if(type[i]==-1) type[i]=0;
            if(!dfs(i, graph, type)) return false;
        }
        return true;
    }
    public boolean dfs(int pos, int[][] graph, int[] type){
        int t=type[pos];
        for(int i=0; i<graph[pos].length; i++){
            int next_node=graph[pos][i];
            if(type[next_node]!=-1){
                if(type[next_node]+t!=1) return false;
            }else{
                type[next_node]=1-t;
                if(!dfs(next_node, graph, type)) return false;
            }
        }
        return true;
    }
}
```

