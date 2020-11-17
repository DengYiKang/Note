# Longest Palindromic Subsequence

### 问题

求最长回文子串，该子串可以不连续。

### 解决方案：dp，时间复杂度$O(n^2)$，空间复杂度$O(n^2)$

如果是连续的子串的话，用一维dp即可，`dp[i]`表示以`str[i]`结尾的最长连续的回文子串。

这里是允许离散的，那么用二维dp来表示，`dp[i][j]`表示`str[i...j]`中最长回文子串。

那么有：

`dp[i][j]=dp[i+1][j-1] (if str[i]==str[j])`

`dp[i][j]=max(dp[i+1][j], dp[i][j-1]) (if str[i]!=str[j])`

```java
class Solution {
    public int longestPalindromeSubseq(String s) {
        if(s==null||s.length()==0) return 0;
        int[][] dp=new int[s.length()][s.length()];
        for(int i=s.length()-1; i>=0; i--){
            dp[i][i]=1;
            for(int j=i+1; j<s.length(); j++){
                if(s.charAt(i)==s.charAt(j)){
                    dp[i][j]=dp[i+1][j-1]+2;
                }else{
                    dp[i][j]=Math.max(dp[i+1][j], dp[i][j-1]);
                }
            }
        }
        return dp[0][s.length()-1];
    }
}
```

