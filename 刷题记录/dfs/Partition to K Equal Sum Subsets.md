# Partition to K Equal Sum Subsets

### 问题

数组`nums`与正整数`k`，判断`nums`能否被划分成`k`个子数组，它们的和相等。

### 解决方案

经典的回溯型dfs。注意在搜索同一个子数组的元素时是从当前`pos`往后搜索的。但开始搜索新的子数组时是从`0`开始搜索。

```java
class Solution {
    public boolean canPartitionKSubsets(int[] nums, int k) {
        int sum=Arrays.stream(nums).sum();
        if(sum%k!=0) return false;
        int target=sum/k;
        boolean[] vis=new boolean[nums.length];
        vis[0]=true;
        return canPartition(0, k, 0, target, nums, vis);
    }
    public boolean canPartition(int pos, int k, int sum, int target,
                                int[] nums, boolean[] vis){
        if(k==1) return true;
        if(sum>target) return false;
        if(sum==target){
            return canPartition(0, k-1, 0, target, nums, vis);
        }
        for(int i=pos+1; i<nums.length; i++){
            if(vis[i]) continue;
            vis[i]=true;
            if(canPartition(i, k, sum+nums[i], target, nums, vis)) {
                return true;
            }
            vis[i]=false;
        }
        return false;
    }
}
```

