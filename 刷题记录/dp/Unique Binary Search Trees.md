# Unique Binary Search Trees

### 问题

输入一个数字n，要求输出存储1-n的所有异构BST的数量。

### 解决方案:dp,$O(n^2)$

dp[n]表示存储1-n的所有异构BST的数量。

考虑根的值为root，则左边有root-1个，右边有n-root个，注意到子树的结点都是不同的，即可视为子问题。

因此有：
$$
dp[n]=\Sigma_{i=0}^{n-1}dp[i]*dp[n-i-1]
$$

```java
class Solution {
    public int numTrees(int n) {
        int[] dp=new int[n+1];
        dp[0]=1;
        for(int i=1; i<=n; i++){
            for(int j=0; j<i; j++){
                dp[i]+=dp[j]*dp[i-j-1];
            }
        }
        return dp[n];
    }
}
```

