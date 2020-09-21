# Maximal Square

### 问题

输入一个二维的01矩阵，求该矩阵中包含的面积最大的元素全为1的方阵的元素个数。

### 解决方案：dp，时间复杂度$O(mn)$，空间复杂度$O(mn)$ 

令$dp[i][j]$表示以$(i,j)$为右下角的面积最大的元素全为1的方阵的边长。那么则有：
$$
dp[0][i]=
\begin{cases}
0,&\mbox{if A[0][i]=0}\\
1,&\mbox{if A[0][i]=1}
\end{cases}\\
dp[i][0]=
\begin{cases}
0,&\mbox{if A[i][0]=0}\\
1,&\mbox{if A[i][0]=1}
\end{cases}\\
dp[i][j]=
\begin{cases}
min{dp[i-1][j],dp[i][j-1],dp[i-1][j-1]}+1,&\mbox{if A[i][j]}\neq0\\
0,&\mbox{if A[i][j] = 0}
\end{cases}
$$

```java
class Solution {
    public int maximalSquare(char[][] matrix) {
        int h=matrix.length;
        if(h==0) return 0;
        int w=matrix[0].length;
        int[][] dp=new int[h][w];
        for(int i=0; i<w; i++){
            dp[0][i]=matrix[0][i]=='1'?1:0;
        }
        for(int i=0; i<h; i++){
            dp[i][0]=matrix[i][0]=='1'?1:0;
        }
        for(int i=1; i<h; i++){
            for(int j=1; j<w; j++){
                if(matrix[i][j]=='0') dp[i][j]=0;
                else{
                    dp[i][j]=Math.min(Math.min(dp[i-1][j], dp[i][j-1]), dp[i-1][j-1])+1;
                }
            }
        }
        int ans=0;
        for(int i=0; i<h; i++){
            for(int j=0; j<w; j++){
                ans=Math.max(ans, dp[i][j]);
            }
        }
        return ans*ans;
    }
}
```

