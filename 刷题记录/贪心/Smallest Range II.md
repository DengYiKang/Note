# Smallest Range II

### 问题

`A`为整型数组，`k`为正整数，对`A`的所有元素进行`+K`或`-K`操作，之后得到数组`B`。求`min(max(B)-min(B))`。

### 解决方案：贪心，时间复杂度$O(n)$

对于两个数`A[i]`与`A[j]`，设`A[i]<A[j]`，那么对于它们之间的差值，`(A[i]+K, A[j]-K)`肯定比`(A[i]-K, A[j]+k)`要小。

那么对`A`进行升序排序，这里有个很难想到的一个点，就是最优的形式是`A[0]+K, A[1]+K, ..., A[i]+K`

`A[i+1]-K, A[i+2]-K, ..., A[N]-K`这种形式。假设在前一段（即`+K`这个段）中某个点`-K`，那么可能会导致最小值会更小，那么将导致`max(B)-min(B)`更大。因此只需搜索这个分界线即可。

并且这种形式里，最大值只可能在`A[i]+K`和`A[N]-K`之间产生，最小值只可能在`A[0]+K`和`A[i+1]-K`中产生。

```java
class Solution {
    public int smallestRangeII(int[] A, int K) {
        int N=A.length;
        Arrays.sort(A);
        int ans=A[N-1]-A[0];
        for(int i=0; i<N-1; i++){
            int a=A[i]+K, b=A[i+1]-K;
            int hi=Math.max(A[N-1]-K, a);
            int lo=Math.min(A[0]+K, b);
            ans=Math.min(ans, hi-lo);
        }
        return ans;
    }
}
```

