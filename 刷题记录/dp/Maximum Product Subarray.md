# Maximum Product Subarray

### 问题

输入一个整形数组，求连续子数组的最大积

### 解决方案：dp，$O(n)$

```java
//l[i]表示以nums[i]结尾的最小积
//h[i]表示以nums[i]结尾的最大积
class Solution {
    public int maxProduct(int[] nums) {
        if(nums==null||nums.length==0) return 0;
        int[] l=new int[nums.length];
        int[] h=new int[nums.length];
        int ans=nums[0];
        l[0]=nums[0];
        h[0]=nums[0];
        for(int i=1; i<nums.length; i++){
            l[i]=Math.min(Math.min(l[i-1]*nums[i], h[i-1]*nums[i]), nums[i]);
            h[i]=Math.max(Math.max(l[i-1]*nums[i], h[i-1]*nums[i]), nums[i]);
            ans=Math.max(ans, h[i]);
        }
        return ans;
    }
}
```

