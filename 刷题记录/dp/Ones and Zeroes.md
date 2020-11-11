# Ones and Zeroes

### 题目

现有集合`strs`，其元素是表示二进制的字符串，两个数`m`和`n`。

求`strs`的最大子集的大小，其中的所有元素里的0,1总数分别不超过`m`和`n`。

### 解决方案

与背包问题差不多。

```java
class Solution {
    public int findMaxForm(String[] strs, int m, int n) {
        if(strs==null||strs.length==0) return 0;
        int len=strs.length;
        int[][][] dp=new int[len+1][m+1][n+1];
        for(int i=0; i<len+1; i++){
            for(int j=0; j<m+1; j++){
                Arrays.fill(dp[i][j], -1);
            }
        }
        int[][] cnt=new int[strs.length][2];
        for(int i=0; i<strs.length; i++){
            for(int j=0; j<strs[i].length(); j++){
                if(strs[i].charAt(j)=='0') cnt[i][0]++;
                else cnt[i][1]++;
            }
        }
        return getDp(len, m, n, cnt, dp);
    }
    public int getDp(int i, int m, int n, int[][] cnt, int[][][] dp){
        if(i<=0||m<=0&&n<=0) return 0;
        if(dp[i][m][n]!=-1) return dp[i][m][n];
        int result=getDp(i-1, m, n, cnt, dp);
        if(m-cnt[i-1][0]>=0&&n-cnt[i-1][1]>=0){
            result=Math.max(result, getDp(i-1, m-cnt[i-1][0], n-cnt[i-1][1], cnt, dp)+1);
        }
        dp[i][m][n]=result;
        return result;
    }
}
```

