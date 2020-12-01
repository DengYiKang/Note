# Permutation in String

### 问题

有两个字符串`s1`，`s2`。问`s2`是否包含`s1`的排列。注意是`s2`的连续子串。

### 解决方案：时间复杂度$O(n)$ 

如果能定义一种广义减法，使得可以通过判断`s2`中的累计状态相减是否能得到`s1`的状态来解决问题，那么可以把该问题转换成连续子序列之和的问题，同时用HashSet来优化成`O(n)`。

这里的状态用字符串来表示，`1_2_3_...`来表示`a`的数量为1，`b`的数量为2等等。

```java
class Solution {
    public boolean checkInclusion(String s1, String s2) {
        if(s1==null||s1.length()==0) return true;
        if(s2==null||s2.length()==0) return false;
        int[] data=new int[26];
        for(char ch:s1.toCharArray()){
            data[ch-'a']++;
        }
        HashSet<String> set=new HashSet<>();
        int[] init=new int[26];
        set.add(getStatus(init));
        int[] sum=new int[26];
        for(char ch:s2.toCharArray()){
            sum[ch-'a']++;
            int[] diff=getDiff(sum, data);
            String need=getStatus(diff);
            if(set.contains(need)) return true;
            set.add(getStatus(sum));
        }
        return false;
    }
    int[] getDiff(int[] a, int[] b){
        int[] ans=new int[26];
        for(int i=0; i<26; i++){
            ans[i]=a[i]-b[i];
        }
        return ans;
    }
    String getStatus(int[] a){
        StringBuilder sb=new StringBuilder();
        for(int i=0; i<a.length; i++){
            if(i!=0) sb.append("_");
            sb.append(a[i]);
        }
        return sb.toString();
    }
    
}
```

