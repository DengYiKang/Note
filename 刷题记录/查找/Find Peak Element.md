# Find Peak Element

### 问题

给一组数组$A$,其中$A[i]\neq A[i+1]$，找到任意一个顶峰$k$，满足$A[k]>A[k-1]\;\&\&\;A[k]>A[k+1]$。默认$A[-1]=A[len]=-\infty$。要求时间复杂度为$O(logn)$

### 解决方案：折半查找，$O(logn)$ 

在搜索的中点处取一小段($A[mid],A[mid+1]$)查看是上坡还是下坡，若是上坡则峰顶在右侧，若是下坡则峰顶在左侧。

```java
class Solution {
    public int findPeakElement(int[] nums) {
        int lo=0, hi=nums.length-1;
        while(lo<hi){
            int mid=(lo+hi)/2;
            int mid2=mid+1;
            //注意lo与hi的赋值
            if(nums[mid]<nums[mid2]) lo=mid2;
            else hi=mid;
        }
        return lo;
    }
}
```

