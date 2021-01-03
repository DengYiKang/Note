# Largest Sum of Averages

### 问题

将数组`A`划分为`K`个相邻元素的组，使这`K`个的平均值累加和最大，求最大累加平均值。

### 解决方案：dp，时间复杂度$O(KN)$

令`dp[n, k]`表示最大累加平均值，则有：`dp[n, k]=max{dp[n-1, k-1]+v1, dp[n-2, k-1]+v2, ...}`

其中`v1， v2, ...`可以用`sum`进行累加计算。

```java
class Solution {
    public double largestSumOfAverages(int[] A, int K) {
        HashMap<Integer, Double> mp=new HashMap<>();
        return getDp(A.length-1, K, A, mp);
    }
    public double getDp(int n, int k, int[] A, HashMap<Integer, Double> mp){
        int code=hashCode(n, k);
        if(mp.containsKey(code)) return mp.get(code);
        if(n+1<k) return Double.MIN_VALUE;
        if(k==1){
            double ans=0;
            for(int i=0; i<=n; i++) ans+=A[i];
            return ans/(n+1);
        }
        double sum=A[n], max_average=-1;
        for(int i=n-1; i>=0; i--){
            max_average=Math.max(max_average, getDp(i, k-1, A, mp)+sum/(n-i));
            sum+=A[i];
        }
        mp.put(code, max_average);
        return max_average;
    }
    public int hashCode(int n, int k){
        return n*101+k;
    }
}
```

