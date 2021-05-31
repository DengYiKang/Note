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

更好理解的版本：

dfs(pos)表示从pos开始匹配，一定要将pos匹配进去。

有两种情况，一是当前组已经匹配完了，需要重新开始匹配，那么对第一个非已匹配状态进行dfs，如果失败，那么肯定就不存在解。

而在当前组匹配的过程中，是属于可回溯的，因此对于下个位置如果匹配失败，可以换个位置继续匹配，知道成功为止。

```java
class Solution {
    public boolean canPartitionKSubsets(int[] nums, int k) {
        int sum=0;
        for(int x:nums){
            sum+=x;
        }
        if(sum%k!=0) return false;
        return dfs(0, nums, 0, sum/k, new boolean[nums.length]);
    }
    boolean dfs(int pos, int[] nums, int sum, int target, boolean[] vis){
        sum+=nums[pos];
        if(sum>target) return false;
        vis[pos]=true;
        if(sum==target){
            boolean ans=true;
            for(int i=0; i<nums.length; i++){
                if(vis[i]) continue;
                ans=dfs(i, nums, 0, target, vis);
                break;
            }
            vis[pos]=false;
            return ans;
        }
        for(int i=pos+1; i<nums.length; i++){
            if(vis[i]) continue;
            if(dfs(i, nums, sum, target, vis)){
                vis[i]=false;
                return true;
            }
        }
        vis[pos]=false;
        return false;
    }
}
```

