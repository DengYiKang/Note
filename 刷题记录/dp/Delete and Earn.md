# Delete and Earn

### 问题

整型数组`nums`，你可以做以下操作：

在每个操作中，你可以从中删除一个数`nums[i]`，你得到`nums[i]`个积分，之后你必须连带删除所有`nums[i]-1`和`nums[i]+1`的数（这些删除无法获得积分）。

问得到的最大的积分是多少？

### 解决方案：dp，时间复杂度$O(n)$

注意到，如果`nums[i]`有多个，你选了它，之后删除它大小相邻的数，那么其余的`nums[i]`大小的数将不会被删去，因此每次可以直接把所有大小为`nums[i]`的数全部取晚。

因此该问题就转换成背包问题了（取或不取）。

```java
class Solution {
    static int MAXN=10001;
    public int deleteAndEarn(int[] nums) {
        int[] count=new int[MAXN];
        for(int num:nums){
            count[num]++;
        }
        int[] dp=new int[MAXN];
        dp[0]=0;
        dp[1]=count[1];
        for(int i=2; i<MAXN; i++){
            dp[i]=Math.max(dp[i-2]+i*count[i], dp[i-1]);
        }
        return dp[MAXN-1];
    }
}
```

