# Gas Station

### 问题

输入两组数组gas[1...n]与cost[1...n]，若$\Sigma_{k=i}^{j}gas[k]-cost[k]>0$表示$i$能到达$j$。

求在哪个起点可以绕一周会原地？

### 解决方案：贪心,$O(n)$

考虑有序列AB...CD。

设A恰好不能到达D，则A必>0，D必<0（否则为A恰好不能到达C）。

对于其子序列，因为A>0,显然sum(A, D)>sum(B, D)，不断递归可知A...D间的点无法到达D。因此若某个序列不符题意，则无需考虑其子序列。

对于循环，设tot表示之前失败的所有和，tank表示某点F成功到达末端的剩余值，若tot+tank<0则无解，反之点F即为解。（反证：设有序列A...B...C...D，C成功到达D点，若tot+tank>=0，且C->D->B失败了，那么B->C一定是成功的，矛盾；若tot+tan<0显然不可能成功）

```java
class Solution {
    public int canCompleteCircuit(int[] gas, int[] cost) {
        int tot=0, tank=0, start=0;
        int len=gas.length;
        for(int i=0; i<len; i++){
            tank+=gas[i]-cost[i];
            if(tank<0){
                start=i+1;
                tot+=tank;
                tank=0;
            }
        }
        return tot+tank<0?-1:start;
    }
}
```

