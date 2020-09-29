# Maximum Product of Word Lengths

### 问题

输入一个字符串数组`words`，找到`length(word[i])*length(word[j])`的最大值，要求`word[i]`与`word[j]`不能拥有相同的字符。

### 解决方案：时间复杂度$O(n)$，空闲复杂度$O(n)$

26种字符，可以用一个`Integer`保存状态，检查是否拥有相同的字符直接进行与运算即可。

```java
class Solution {
    public int maxProduct(String[] words) {
        if(words==null||words.length==0) return 0;
        int[] flag=new int[words.length];
        for(int i=0; i<words.length; i++){
            int[] bit=new int[26];
            for(int j=0; j<words[i].length(); j++){
                int id=words[i].charAt(j)-'a';
                bit[id]=1;
            }
            for(int j=0; j<26; j++){
                flag[i]=flag[i]<<1;
                flag[i]+=bit[j];
            }
        }
        int maxn=0;
        for(int i=0; i<words.length; i++){
            for(int j=i+1; j<words.length; j++){
                if(i==j||(flag[i]&flag[j])!=0) continue;
                maxn=Math.max(maxn, words[i].length()*words[j].length());
            }
        }
        return maxn;
    }
}
```

