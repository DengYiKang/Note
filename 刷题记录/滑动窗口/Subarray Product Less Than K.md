# Subarray Product Less Than K

### 问题

`nums`为正整数数组，计算连续积小于`k`的连续子数组的个数。

### 解决方案：滑动窗口，时间复杂度$O(n)$ 

`lo， hi`分别维护滑动窗口的两端，每次移动右端，如果当前连续积大于等于`k`，则移动左端，左端移动一位，则`nums[lo]`离开窗口，那么需要累加包括`nums[lo]`在内的原窗口中的连续子数组的个数，即`hi-lo`（注意`hi`是恰好使窗口的值大于或等于`k`）。

```java
class Solution {
    public int numSubarrayProductLessThanK(int[] nums, int k) {
        int lo=0, hi=0;
        int ans=0;
        long product=1;
        while(lo<=hi&&hi<nums.length){
            product*=nums[hi];
            while(lo<=hi&&product>=k){
                product/=nums[lo];
                ans+=hi-lo;
                lo++;
            }
            hi++;
        }
        //注意乘积的地方可能溢出
        return ans+=((long) hi-lo+1)*((long) hi-lo)/2;
    }
}
```

