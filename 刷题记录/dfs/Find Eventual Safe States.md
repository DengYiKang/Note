#  Find Eventual Safe States

### 问题

有向图`Graph`，一个结点是安全的当且仅当从该节点出发，选择任意的路径都能到达一个DeadEnd。返回这些安全的结点，升序。

### 解决方案：时间复杂度$O(Edges)$

如果存在环，那么肯定不是安全的，可以在dfs中加入回溯的vis状态。

同时也加入safe和notSafe状态来进行记忆化搜索。

```java
class Solution {
    public List<Integer> eventualSafeNodes(int[][] graph) {
        HashSet<Integer> notSafe=new HashSet<>();
        HashSet<Integer> safe=new HashSet<>();
        HashSet<Integer> vis=new HashSet<>();
        for(int i=0; i<graph.length; i++){
            if(graph[i].length==0) safe.add(i);
        }
        List<Integer> ans=new ArrayList<>();
        for(int i=0; i<graph.length; i++){
            if(dfs(i, graph, notSafe, safe, vis)) ans.add(i);
        }
        return ans;
    }
    public boolean dfs(int node, int[][] graph, HashSet<Integer> notSafe, HashSet<Integer> safe, HashSet<Integer> vis){
        if(vis.contains(node)) return false;
        if(safe.contains(node)) return true;
        if(notSafe.contains(node)) return false;
        //注意vis的位置，不要放在上面的return语句之中或之前
        vis.add(node);
        for(int i=0; i<graph[node].length; i++){
            if(!dfs(graph[node][i], graph, notSafe, safe, vis)){
                notSafe.add(node);
                return false;
            }
        }
        vis.remove(node);
        safe.add(node);
        return true;
    }
}
```

