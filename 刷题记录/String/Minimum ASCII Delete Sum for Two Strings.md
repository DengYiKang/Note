# Minimum ASCII Delete Sum for Two Strings

### 问题

有两个字符串`s1, s2`，在两个字符串里面删去一些字符使得两个字符串相同，求删去的这些字符的ASCII码之和的最小值。

### 解决方案：dp，时间复杂度$O(mn)$

`Delete Operation for Two String`的问题与此问题类似，前者可以看做权值为1，后者可以看做权值为对应的ASCII码。前者的权值为1的语义是计算最长的相同子序列的问题。注意到，对于此问题，正确的结果必然是在最长的相同子序列的集合中，因此可以考虑使用前者的动态规划思路，改变权值即可。

```java
class Solution {
    public int minimumDeleteSum(String s1, String s2) {
        char[] a=s1.toCharArray();
        char[] b=s2.toCharArray();
        int sum1=0, sum2=0;
        for(int i=0; i<a.length; i++){
            sum1+=a[i];
        }
        for(int i=0; i<b.length; i++){
            sum2+=b[i];
        }
        int[][] dp=new int[a.length][b.length];
        for(int i=0; i<dp.length; i++){
            Arrays.fill(dp[i], -1);
        }
        for(int i=0; i<a.length; i++){
            if(a[i]==b[0]){
                while(i<a.length){
                    dp[i++][0]=b[0];
                }
            }else{
                dp[i][0]=0;
            }
        }
        for(int i=0; i<b.length; i++){
            if(b[i]==a[0]){
                while(i<b.length){
                    dp[0][i++]=a[0];
                }
            }else{
                dp[0][i]=0;
            }
        }
        return sum1+sum2-2*getDp(a.length-1, b.length-1, a, b, dp);
    }
    public int getDp(int i, int j, char[] a, char[] b, int[][] dp){
        if(dp[i][j]!=-1) return dp[i][j];
        int ans=0;
        if(a[i]==b[j]) ans=getDp(i-1, j-1, a, b, dp)+a[i];
        ans=Math.max(ans, getDp(i-1, j, a, b, dp));
        ans=Math.max(ans, getDp(i, j-1, a, b, dp));
        dp[i][j]=ans;
        return ans;
    }
}
```

