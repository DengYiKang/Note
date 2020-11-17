# Longest Uncommon Subsequence II

### 问题

有一个字符串集`String[] strs`，找出最长非公共子串（不要求连续），即该串仅为`str[i]`的子串，不是其他字符串的子串。输出最长非公共子串的长度，若无非公共子串，则输出-1。

### 解决方案

注意到，如果存在非公共子串，那么最长非公共子串必然是某个`str[i]`它本身，因为若其本身不符合，那么它的任意子串也不符合。

因此，将`strs[1...n]`按其串的长度从大到小排列，若当前串没有重复过且不被任意其它更长的串包含，那么该串本身就是最长非公共子串。

```java
class Solution {
    public int findLUSlength(String[] strs) {
        Arrays.sort(strs, (a, b)->(b.length()-a.length()));
        HashSet<String> duplicates=getDuplicates(strs);
        for(int i=0; i<strs.length; i++){
            if(duplicates.contains(strs[i])) continue;
            if(i==0) return strs[i].length();
            for(int j=0; j<i; j++){
                if(isSubsequence(strs[i], strs[j])) break;
                //这种方式我觉得很好用，以前我一般置flag的
                if(j==i-1) return strs[i].length();
            }
        }
        return -1;
    }
    boolean isSubsequence(String a, String b){
        int i=0, j=0;
        while(i<a.length()&&j<b.length()){
            if(a.charAt(i)==b.charAt(j)) i++;
            j++;
        }
        return i==a.length();
    }
    HashSet<String> getDuplicates(String[] strs){
        HashSet<String> set=new HashSet<>();
        HashSet<String> duplicates=new HashSet<>();
        for(String s:strs){
            if(set.contains(s)){
                duplicates.add(s);
            }else{
                set.add(s);
            }
        }
        return duplicates;
    }
}
```



