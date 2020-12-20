# Network Delay Time

### 问题

一个有向图，边有对应的权值，一个信号从某个结点发出，求问所有结点是否能收到信号，如果能，需要多久所有结点才能接受到信号？

### 解决方案：bfs，时间复杂度$O(n)$ 

优先队列，并且注意后续加入队列的结点可能与队列中已有的结点相同，但其所带的时间戳更前，因此注意`vis`设置的时间，应该在出对的时候设置，而不是入队的时候设置。

```java
class Solution {
    public int networkDelayTime(int[][] times, int N, int K) {
        int cnt=0;
        int[][] graph=new int[N+1][N+1];
        for(int i=0; i<graph.length; i++){
            Arrays.fill(graph[i], -1);
        }
        for(int[] term:times){
            graph[term[0]][term[1]]=term[2];
        }
        PriorityQueue<int[]> queue=new PriorityQueue<>((a, b)->a[1]-b[1]);
        boolean[] vis=new boolean[N+1];
        queue.offer(new int[]{K, 0});
        int max_time=0;
        while(!queue.isEmpty()){
            int[] node=queue.poll();
            if(vis[node[0]]) continue;
            //注意是出对后设置vis
            vis[node[0]]=true;
            cnt++;
            max_time=node[1];
            for(int i=1; i<N+1; i++){
                if(!vis[i]&&graph[node[0]][i]!=-1){
                    queue.offer(new int[]{i, node[1]+graph[node[0]][i]});
                }
            }
        }
        return cnt==N?max_time:-1;
    }
}
```

