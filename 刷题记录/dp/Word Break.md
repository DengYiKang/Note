# Word Break

### 问题

输入一个字符串s与一个字典，问字典中的字符串组合成s？

### 解决方案：DP，$O(N^2)$

设$dp[i]$表示$s[0...i]$能否被字典中的字符串组合，则可以如下计算$dp[i]$：

```java
dp[i]=dict.contains(s[0...i])|dp[0]&&dict.contains(s[1...i])|...
    |dp[i-1]&&dict.contains(s[i]);
```

```java
class Solution {
    
    public boolean wordBreak(String s, List<String> wordDict) {
        HashSet<String> dict=new HashSet<>();
        for(String str:wordDict){
            dict.add(str);
        }
        int len=s.length();
        boolean[] dp=new boolean[len];
        dp[0]=dict.contains(String.valueOf(s.charAt(0)));
        for(int i=1; i<len; i++){
            for(int j=0; j<i; j++){
                if(!dp[j]) continue;
                if(dict.contains(s.substring(j+1,i+1))){
                    dp[i]=true;
                    break;
                }
            }
            if(dict.contains(s.substring(0,i+1))) dp[i]=true;
        }
        return dp[len-1];
    }
}
```

