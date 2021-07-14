# Is Graph Bipartite?

### 问题

判断一个图是否为二分图。

### 解决方案：dfs，时间复杂度$O(n)$

dfs进行染色，染色数组`int[] color`即可以表示当前的染色状态又可以表示是否被dfs过。

```java
class Solution {
    public boolean isBipartite(int[][] graph) {
        if(graph==null||graph.length==0) return false;
        int n=graph.length;
        int[] colors=new int[n];
        for(int i=0; i<graph.length; i++){
            if(colors[i]==0&&!dfs(i, graph, colors, 1)) return false;
        }
        return true;
    }
    boolean dfs(int pos, int[][] graph, int[] colors, int color){
        colors[pos]=color;
        int next_color=-color;
        for(int i=0; i<graph[pos].length; i++){
            int next_node=graph[pos][i];
            if(colors[next_node]==color) return false;
            if(colors[next_node]==0&&!dfs(next_node, graph, colors, next_color)){
                return false;
            }
        }
        return true;
    }
}
```

