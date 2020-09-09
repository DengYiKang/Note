# Unique Paths

### 问题

输入一个m*n的网格，求左上顶点到右下顶点的路径数。

### 解决方案

很容易想到用DP做

```java
    public int uniquePaths(int m, int n) {
        int[][] dp=new int[m][n];
        for(int i=0; i<m; i++){
            dp[i][0]=1;
        }
        for(int i=0; i<n; i++){
            dp[0][i]=1;
        }
        for(int i=1; i<m; i++)
            for(int j=1; j<n; j++)
                dp[i][j]=dp[i-1][j]+dp[i][j-1];
        return dp[m-1][n-1];
    }
```

可以注意到，每次计算$dp[i][j]$时，只用到上一行$dp[i-1][j]$和当前行$dp[i][j-1]$。

因此可以对空间进行优化：

```c++
//因为java里没swap，所以就偷懒了，把c++代码搬来了
class Solution {
public:
    int uniquePaths(int m, int n) {
        vector<int> pre(n, 1), cur(n, 1);
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                cur[j] = pre[j] + cur[j - 1];
            }
            swap(pre, cur);
        }
        return pre[n - 1];
    }
};
```

注意到，pre[j]在更新之前就是cur[j]，因此继续优化：

```c++
class Solution {
public:
    int uniquePaths(int m, int n) {
        vector<int> cur(n, 1);
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                cur[j] += cur[j - 1];
            }
        }
        return cur[n - 1];
    }
};
```

