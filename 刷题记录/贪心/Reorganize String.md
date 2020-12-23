# Reorganize String

### 问题

`S`是一个只包含26个小写字母的字符串，要求对它进行重排，使得相邻的字符不同。输出任意一个合法排列，如果不存在则输出空串。

### 解决方案：贪心，时间复杂度$O(n)$ 

先取频数最大的字符进行排列，其他字符对其进行插空。假设频数最大的字符为`a`：`a#a#a...a#a#`，注意最后一个字符`a`后面也要进行插空，否则如果存在多个最大频数的字符，将无法满足题意。相反地，如果把频数最大的字符排列后，设其频数为`m`，剩下的字符数为`n-m`，如果有`m-1>n-m`，则不存在。

有两种方式实现插空，一种时构造`m`个`StringBuilder`进行轮流`append`；一种是只用一个`StringBuilder`，对当前的划分（指以`a`为首、下一个`a`的前个字符为尾）用定位的方式进行`append`。

这里用的是第二种实现：

```java
class Solution {
    public String reorganizeString(String S) {
        char head='#';
        int max_cnt=0;
        int[] cnt=new int[26];
        char[] chs=S.toCharArray();
        for(char ch:chs){
            cnt[ch-'a']++;
        }
        for(int i=0; i<26; i++){
            if(max_cnt<cnt[i]){
                max_cnt=cnt[i];
                head=(char)('a'+i);
            }
        }
        if(max_cnt-1>S.length()-max_cnt) return "";
        char[] less=new char[S.length()-max_cnt];
        int tail=0;
        for(int i=0; i<26; i++){
            char ch=(char)('a'+i);
            if(ch==head) continue;
            for(int j=0; j<cnt[i]; j++){
                less[tail++]=ch;
            }
        }
        StringBuilder ans=new StringBuilder();
        for(int i=0; i<max_cnt; i++){
            StringBuilder sb=new StringBuilder();
            sb.append(head);
            for(int j=0; max_cnt*j+i<less.length; j++){
                sb.append(less[max_cnt*j+i]);
            }
            ans.append(sb.toString());
        }
        return ans.toString();
    }
}
```

