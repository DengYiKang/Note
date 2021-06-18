# Longest Substring with At Least K Repeating Characters

### 问题

现有字符串`s`，整数`k`，要求找到一个这样的子串，其所有的元素的出现次数大于等于`k`，求这种子串的最大长度。

### 解决方案

```java
class Solution {
    public int longestSubstring(String s, int k) {
        if(s==null||s.length()==0) return 0;
        char[] str=s.toCharArray();
        //每个字母的计数
        int[] cnt=new int[26];
        for(int i=0; i<str.length; i++){
            int id=str[i]-'a';
            cnt[id]++;
        }
        boolean valid=true;
        for(int i=0; i<26; i++){
            if(cnt[i]<k&&cnt[i]>0) valid=false;
        }
        //整个字符串符合题意
        if(valid) return str.length;
        int left=0, right=0, ans=0;
        while(right<str.length){
            int id=str[right]-'a';
            if(cnt[id]<k){
                //递归
                ans=Math.max(ans, longestSubstring(s.substring(left, right), k));
                left=right+1;
                right++;
            }else{
                right++;
            }
        }
        ans=Math.max(ans, longestSubstring(s.substring(left, right), k));
        return ans;
    }
}
```

某个字符如果总数不超过k，那么合法区间必然不会跨越它，那么可以根据这个边界来分治：

```java
class Solution {
    public int longestSubstring(String s, int k) {
        return longestSubstringUtil(s, 0, s.length(), k);
    }

    int longestSubstringUtil(String s, int start, int end, int k) {
        if (end < k) return 0;
        int[] countMap = new int[26];
        // update the countMap with the count of each character
        for (int i = start; i < end; i++)
            countMap[s.charAt(i) - 'a']++;
        for (int mid = start; mid < end; mid++) {
            if (countMap[s.charAt(mid) - 'a'] >= k) continue;
            int midNext = mid + 1;
            while (midNext < end && countMap[s.charAt(midNext) - 'a'] < k) midNext++;
            return Math.max(longestSubstringUtil(s, start, mid, k),
                    longestSubstringUtil(s, midNext, end, k));
        }
        return (end - start);
    }
}
```

