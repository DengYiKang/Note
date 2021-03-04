# Minimum Cost For Tickets

### 问题

`int[] days`表示一年中计划旅行的天的序号，为严格递增的。`int[] cost`为旅行特定的天数的花费。`cost[0]`表示旅行一天的花费，`cost[1]`表示连续旅行7天的花费（不一定要旅行满），`cost[2]`表示连续旅行30天的花费。

问能覆盖`days`的最小花费是多少。

### 解决方案：dp，时间复杂度$O(38N)$

```
dp[i]=min(
dp[i-1]+cost[0],
min{dp[j]+cost[1] | dp[i]-dp[j+1]<7}, 
min{dp[j]+cost[2] | dp[i]-dp[j+1]<30}
)
```

要注意，因为`days[0]`可能会合并到后面的计划中，在计算中会引入`dp[-1]`，因此需要引入dummy node。

```java
class Solution {
        public int mincostTickets(int[] days, int[] costs) {
                int N=days.length;
            	//注意要引入dummy node
                int[] dp=new int[N+1];
                for(int i=1; i<N+1; i++){
                        int min_cost=costs[0]+dp[i-1];
                        int index=i-1;
                        int j=index;
                        while(j>=0&&days[index]-days[j]<7){
                                min_cost=Math.min(min_cost, dp[j]+costs[1]);
                                j--;
                        }
                        j=index;
                        while(j>=0&&days[index]-days[j]<30){
                                min_cost=Math.min(min_cost, dp[j]+costs[2]);
                                j--;
                        }
                        dp[i]=min_cost;
                }
                return dp[N];
        }
}
```

