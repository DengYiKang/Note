# Delete Operation for Two Strings

### 问题

有两个字符串`word1, word2`，每步将`word1`或`word2`中的某个字符删去，找到最少的步数使得两个字符串相同。

### 解决方案：dp，时间复杂度$O(mn)$

设经过`k`步使得两个字符串相同，其长度为`m`，那么有`k+2*m=n`，其中`n`是两个字符串的长度之和。要使`k`最小，那么就要使`m`最大，问题就转换成求两个字符串的最长子序列的问题。

设`dp[i][j]`表示`word1[0...i]`与`word2[0...j]`之间的最长子序列长度。

计算`dp[i][j]`可分为以下二种情况：

+ `word1[i]==word2[j]`：那么`dp[i-1][j-1]+1`是其候选值。
+ `word1[i]!=word2[j]`：那么`dp[i-1][j]`与`dp[i][j-1]`是其候选值。

在`word1[i]==word2[j]`的这种情况下，如果对`dp[i-1][j-1]+1`与`dp[i-1][j], dp[i][j-1]`的大小不确定（因为其实在`word1[i]==word2[j]`以及不考虑`dp[i][j]`最大的情况下，确实可以取`dp[i-1][j]`或`dp[i][j-1]`的值），那么可以直接取三者最大值。

```java
class Solution {
    int[][] dp;
    public int minDistance(String word1, String word2) {
        if(word1==null||word1.length()==0||word2==null||word2.length()==0){
            return word1.length()+word2.length();
        }
        dp=new int[word1.length()+1][word2.length()+1];
        for(int i=0; i<dp.length; i++){
            for(int j=0; j<dp[0].length; j++){
                dp[i][j]=-1;
            }
        }
        char[] s1=word1.toCharArray();
        char[] s2=word2.toCharArray();
        //对0索引的初始化
        for(int i=0; i<s2.length; i++){
            if(s2[i]==s1[0]){
                while(i<s2.length){
                    dp[0][i]=1;
                    i++;
                }
            }else{
                dp[0][i]=0;
            }
        }
        for(int i=0; i<s1.length; i++){
            if(s1[i]==s2[0]){
                while(i<s1.length){
                    dp[i][0]=1;
                    i++;
                }
            }else{
                dp[i][0]=0;
            }
        }
        return s1.length+s2.length-2*getDp(s1.length-1, s2.length-1, s1, s2);
    }
    public int getDp(int i, int j, char[] s1, char[] s2){
        if(dp[i][j]!=-1) return dp[i][j];
        if(s1[i]==s2[j]){
            dp[i][j]=getDp(i-1, j-1, s1, s2)+1;
        }else{
            dp[i][j]=Math.max(getDp(i-1, j, s1, s2), getDp(i, j-1, s1, s2));
        }
        return dp[i][j];
    }
}
```

