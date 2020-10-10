# Guess Number Higher or Lower II

### 问题

猜数游戏。每次猜错会被告知是高还是低，循环至猜对为止。每次猜错有惩罚，惩罚为当前猜数的罚金。问现输入一个`n`，表示正确数字在`1~n`这个范围，求所有情况下需要准备的罚金的最小数目。

### 解决方案：dp，时间复杂度$O(n^3)$，空间复杂度$O(n^2)$

设`dp[i][j]`表示猜数在区间`[i,j]`这个情况下需要准备的罚金的最小数目，则有：
$$
dp[i][j]=min\{k+max\{dp[i][k-1],dp[k+1,j]\}|k\in[i,j]\}
$$

```java
class Solution {
    int[][] dp;
    public int getMoneyAmount(int n) {
        dp=new int[n+1][n+1];
        for(int i=0; i<n+1; i++){
            Arrays.fill(dp[i], -1);
        }
        for(int i=0; i<n+1; i++){
            dp[i][i]=0;
        }
        return getDp(1, n);
    }
    int getDp(int lo, int hi){
        if(hi<lo) return 0;
        if(dp[lo][hi]!=-1) return dp[lo][hi];
        int max_val=Integer.MAX_VALUE;
        for(int mid=lo; mid<=hi; mid++){
            int tmp=mid+Math.max(getDp(lo, mid-1), getDp(mid+1, hi));
            max_val=Math.min(tmp, max_val);
        }
        dp[lo][hi]=max_val;
        return dp[lo][hi];
    }
}
```

