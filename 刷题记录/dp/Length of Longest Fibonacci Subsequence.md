# Length of Longest Fibonacci Subsequence

### 问题

`A`为严格递增的数列，求`arr`中最长的斐波那契子序列的长度。（在`arr`中可以不连续）

### 解决方案：dp，时间复杂度$O(n^2)$

将`(i, j), i<j`看做一个node，那么如果有`A[i]+A[j]=A[k]`，那么`(i, j)`与`(j, k)`相邻。

设`dp[(i, j)]`表示以`(i, j)`结尾的最长斐波那契子序列长度。因为是严格递增的，所以有：

`dp[(j, k)]=dp[(i, j)]+1, 其中A[i]+A[j]=A[k]`。

和式`A[i]+A[j]=A[k]`可以用`hashmap`来优化。

```java
class Solution {
    public int lenLongestFibSubseq(int[] arr) {
        int N=arr.length;
        HashMap<Integer, Integer> index=new HashMap<>();
        HashMap<Integer, Integer> longest=new HashMap<>();
        for(int i=0; i<arr.length; i++){
            index.put(arr[i], i);
        }
        int ans=0;
        for(int j=0; j<arr.length; j++){
            for(int k=j+1; k<arr.length; k++){
                int i=index.getOrDefault(arr[k]-arr[j], -1);
                if(i>=0&&i<j){
                    //注意这个默认值为2
                    int pre=longest.getOrDefault(i*N+j, 2);
                    longest.put(j*N+k, pre+1);
                    ans=Math.max(ans, pre+1);
                }
            }
        }
        return ans>=3?ans:0;
    }
}
```



