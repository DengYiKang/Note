# Partition Equal Subset Sum

### 问题

现有一个数组`nums`，其元素均为正整型，判断是否有一个划分，使得两个集合中的元素和相等。

### 解决方案

#### 记忆化dfs：时间复杂度$O(2^n)$，空间复杂度$O(n*m)$，其中$m$是元素总和的一半

对于dfs的状态进行记忆：

```java
class Solution {
    Boolean[][] memo;
    public boolean canPartition(int[] nums) {
        int tot=0;
        for(int i=0; i<nums.length; i++) tot+=nums[i];
        if(tot%2!=0) return false;
        memo=new Boolean[nums.length+1][tot/2+1];
        return dfs(0, nums, tot/2);
    }
    boolean dfs(int pos, int[] nums, int sum){
        if(sum<0) return false;
        if(sum==0) return true;
        if(pos>=nums.length) return false;
        if(memo[pos][sum]!=null) return memo[pos][sum];
        boolean result=dfs(pos+1, nums, sum-nums[pos])
            ||dfs(pos+1, nums, sum);
        memo[pos][sum]=result;
        return result;
    }
}
```

#### dp:时间复杂度$O(n*m)$，空间复杂度$O(n*m)$，其中$m$是元素总和的一半

思路与记忆化dfs类似，不过记忆化dfs是从上到下，而dp是从下到上。

```java
class Solution {
    public boolean canPartition(int[] nums) {
        int totalSum=0;
        for (int num:nums) {
            totalSum+=num;
        }
        if(totalSum%2!=0) return false;
        int subSetSum=totalSum/2;
        int n=nums.length;
        boolean dp[][]=new boolean[n+1][subSetSum+1];
        dp[0][0]=true;
        for(int i=1; i<=n; i++){
            int curr=nums[i-1];
            for(int j=0; j<=subSetSum; j++){
                if(j<curr)
                    dp[i][j]=dp[i-1][j];
                else
                    //跟dfs思路一致
                    dp[i][j]=dp[i-1][j]||(dp[i-1][j-curr]);
            }
        }
        return dp[n][subSetSum];
    }
}
```

