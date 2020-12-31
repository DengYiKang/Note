# Number of Subarrays with Bounded Maximum

### 问题

正数元素的整型数组`A`，两个正数`L<=R`，求`A`的所有连续子数组，它们的最大数位于`[L, R]`。

### 解决方案：滑动窗口，时间复杂度$O(n)$

对当前的数`A[i]`，如果它大于`R`，那么肯定不能够组成合法的子数组；如果小于或等于`R`，那么`A[i]`可以跟前面相邻的数进行组合：

+ 如果`L<=A[i]<=R`，那么`A[i]`最远能跟`A[j]`之间的数相组合，其中`j<i, A[j] in [0, R], A[j-1]>R`。即`A[j-1]`是距离`A[i]`最近的比`R`大的数。
+ 如果`A[i]<L`，那么以它为末端的区间必须包含比`L`大的数，左端的极限情况同上。

通过以上分析，我们需要维护一个区间`[lo, hi]`，它的最大数位于`[L, R]`之间，且末位也位于`[L, R]`之间。

```java
class Solution {
    public int numSubarrayBoundedMax(int[] A, int L, int R) {
        int lo=-1, hi=-1, ans=0;
        for(int i=0; i<A.length; i++){
            if(A[i]>R){
                lo=hi=i;
                continue;
            }
            else if(A[i]>=L){
                hi=i;
            }
            ans+=hi-lo;
        }
        return ans;
    }
}
```

