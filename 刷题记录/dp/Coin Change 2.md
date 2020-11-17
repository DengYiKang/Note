# Coin Change 2

### 问题

找钱问题，`amount`为目的金额，`coins[1...n]`为面值类型。假定`coins`的可取数量无上限。问有多少种方案。

### 解决方案：dp，时间复杂度$O(n^2)$，空间复杂度$O(n^2)$

多重背包问题。

```java
class Solution {
    int ans;
    public int change(int amount, int[] coins) {
        //dp[i][j]表示考虑前i种coins(不一定取第1~i种coin)，总和为j的方案总数
        int[][] dp=new int[coins.length+1][amount+1];
        dp[0][0]=1; 
        for(int i=1; i<=coins.length; i++){
            dp[i][0]=1;
            for(int j=1; j<=amount; j++){
                //划分，第i种coin取或不取
                dp[i][j]=dp[i-1][j]+(j-coins[i-1]>=0?dp[i][j-coins[i-1]]:0);
            }
        }
        return dp[coins.length][amount];
    }
}
```



