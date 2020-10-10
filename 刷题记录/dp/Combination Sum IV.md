# Combination Sum IV

### 问题

现有一个无重复整型数组`nums`，给定一个整数`target`，要求和为`target`所有可能的组合。

**Example:**

```bash
nums = [1, 2, 3]
target = 4

The possible combination ways are:
(1, 1, 1, 1)
(1, 1, 2)
(1, 2, 1)
(1, 3)
(2, 1, 1)
(2, 2)
(3, 1)

Note that different sequences are counted as different combinations.
Therefore the output is 7.
```

###  解决方案：dp

注意顺序化。只考虑第一位的数，后面的交于子dp来计算。之前没有考虑顺序化而一直在想某个数在某个区间插空的组合......

```java
class Solution {
    int[] dp;
    public int combinationSum4(int[] nums, int target) {
        if(nums==null||nums.length==0) return 0;
        dp=new int[target+1];
        Arrays.fill(dp, -1);
        dp[0]=1;
        return getDp(target, nums);
    }
    int getDp(int target, int[] A){
        if(dp[target]!=-1) return dp[target];
        int ans=0;
        for(int i=0; i<A.length; i++){
            if(A[i]<=target) ans+=getDp(target-A[i], A);
        }
        dp[target]=ans;
        return dp[target];
    }
}
```



