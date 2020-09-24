# Ugly Number II

### 问题

找到第n个`ugly number`，`ugly number`的素因子只包含`2,3,5`，定义1也是`ugly number`

### 解决方案

该问题可转化为：对于表达式$2^i*3^j*5^k$，如何找到它的从小到大值的排列？

```java
class Solution {
    public int nthUglyNumber(int n) {
        if(n==1) return 1;
        int[] dp=new int[n];
        dp[0]=1;
        int k2=0, k3=0, k5=0;
        for(int i=1; i<n; i++){
            dp[i]=Math.min(dp[k2]*2, Math.min(dp[k3]*3, dp[k5]*5));
            if(dp[i]==dp[k2]*2) k2++;
            if(dp[i]==dp[k3]*3) k3++;
            if(dp[i]==dp[k5]*5) k5++;
        }
        return dp[n-1];
    }
}
```

