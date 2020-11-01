# Unique Substrings in Wraparound String

### 问题

`s`是一个无限循环的字符串，其循环体为`a~z`。

例如`...zabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyza....`。

现有一个字符串`p`，问`p`中有多少**不同**的子串出现在`s`里面。

### 解决方案

对于连续的字符串`abcdefg`，其子串为`n*(n-1)/2+n`，`n`为字符串长度。

那么对它们进行分类，发现可以分为：以某个字母为首的所有子串，或者以某个字母为尾的所有子串。

可以发现，以某个字母为首（尾）的子串的个数等于以该字母为首（尾）的子串的最大长度。

对于重叠问题，我们只需考虑最大长度的子串即可。

而对“以某个字母为尾的最大子串的长度”的统计会更方便。

```java
class Solution {
    public int findSubstringInWraproundString(String p) {
        if(p==null||p.length()==0) return 0;
        char[] str=p.toCharArray();
        int maxLength=1;
        int[] cnt=new int[26];
        cnt[str[0]-'a']=1;
        for(int i=1; i<str.length; i++){
            if((str[i-1]-'a'+1)%26==str[i]-'a') maxLength++;
            else{
                maxLength=1;
            }
            cnt[str[i]-'a']=Math.max(cnt[str[i]-'a'], maxLength);
        }
        int ans=0;
        for(int count:cnt){
            ans+=count;
        }
        return ans;
    }
}
```

