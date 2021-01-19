# Maximum Sum Circular Subarray

### 问题

`A`为整型数组，将`A`首尾相接拼成一个环，在环上取连续的序列，求其最大和是多少？

### 解决方案：dp，时间复杂度$O(n)$，空间复杂度$O(1)$

未拼成环前，设`dp[i]`表示`A`中以`A[i]`结尾的最大和，那么有`dp[i+1]=A[i+1]+max(dp[i], 0)`。

拼成环后，仍可以转换为线性问题。设总和为sum，那么`sum(i, j)+sum(j, i)=sum`。

```java
class Solution {
    public int maxSubarraySumCircular(int[] A) {
        int sum=0, cur_max=0, cur_min=0, ans=Integer.MIN_VALUE;
        for(int x:A){
            sum+=x;
        }
        for(int i=0; i<A.length; i++){
            cur_max=A[i]+Math.max(cur_max, 0);
            cur_min=A[i]+Math.min(cur_min, 0);
            ans=Math.max(ans, cur_max);
            //当i==A.length-1时，在cur_max的情况中已经被计算了，即没有跨边界的情况
            //同时防止cur_min对应整个数组A使得计算非法的长度为0的连续序列
            if(i!=A.length-1) ans=Math.max(ans, sum-cur_min);
        }
        return ans;
    }
}
```

