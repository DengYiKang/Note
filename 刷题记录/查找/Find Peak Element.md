# Find Peak Element

### 问题

给一组数组$A$,其中$A[i]\neq A[i+1]$，找到任意一个顶峰$k$，满足$A[k]>A[k-1]\;\&\&\;A[k]>A[k+1]$。默认$A[-1]=A[len]=-\infty$。要求时间复杂度为$O(logn)$

### 解决方案：折半查找，$O(logn)$ 

套用查找连续相同值的第一个。

那么需要找到一个评判标准，这个评判标准使得划分不重复，且答案在评判标准之内。

那么可以这样定义isRight，如果当前数>下一个数，那么就是isRight，对于峰顶显然是符合的。

```java
class Solution {
    public int findPeakElement(int[] nums) {
        return search(nums, 0, nums.length-1);
    }
    int search(int[] nums, int l, int r){
        while(l<=r){
            int mid=(l+r)>>1;
            if(isRight(nums, mid)){
                r=mid-1;
            }else{
                l=mid+1;
            }
        }
        return l;
    }
    boolean isRight(int[] nums, int pos){
        if(pos==nums.length-1) return true;
        return nums[pos]>nums[pos+1];
    }
}
```



