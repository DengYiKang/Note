# Palindromic Substrings

### 问题

求字符串s中的连续的回文子串的数量。

### 解决方案

#### 一、dp，耗时222ms

设`dp[i][j]`表示`s[i...j]`的回文子串的数量，则有:

`dp[i][j]=dp[i+1][j]+dp[i][j-1]-dp[i+1][j-1]+(s[i...j] is Palindromic)?1:0;`

 ```java
class Solution {
    HashMap<Integer, Integer> mp=new HashMap<>();
    public int countSubstrings(String s) {
        if(s==null||s.length()==0) return 0;
        mp.clear();
        return getDp(0, s.length()-1, s.toCharArray());
    }
    public int getDp(int left, int right, char[] str){
        int code=hashCode(left, right);
        if(mp.containsKey(code)) return mp.get(code);
        if(left>right||left<0||right>=str.length) return 0;
        if(left==right) return 1;
        int val=getDp(left+1, right, str)+getDp(left, right-1, str)-getDp(left+1, right-1, str);
        if(isValid(left, right, str)) val++;
        mp.put(code, val);
        return val;
    }
    public int hashCode(int left, int right){
        return left*1001+right;
    }
    public boolean isValid(int left, int right, char[] str){
        while(left<right){
            if(str[left++]!=str[right--]) return false;
        }
        return true;
    }
}
 ```

#### 二、从中心扩张回文，耗时1ms

```java
class Solution {
    
    public int countSubstrings(String s) {
        char[] str=s.toCharArray();
        int count=0;
        for(int i=0; i<str.length; i++){
            //扩张偶数长度
            count+=extendPalindromic(i, i, str);
            //扩张奇数长度
            count+=extendPalindromic(i, i+1, str);
        }
        return count;
    }
    public int extendPalindromic(int left, int right, char[] str){
        int count=0;
        while(left>=0&&right<str.length&&str[left--]==str[right++]){
            count++;
        }
        return count;
    }
}
```



