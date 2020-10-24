# Longest Repeating Character Replacement

### 问题

现有一个字符串`s`，由大写字母组成，你可以把其中的k个字母替换成任意一个。进行k次操作后，求最长子串的长度，使得该子串由同一个字母组成。

### 解决方案：时间复杂度$O(n)$，空间复杂度$O(n)$

区间是连续的，因此考虑滑动窗口。我们假定滑动窗口通过k次操作始终可以由同一个字母填满。

现假设一个窗口`[left, right]`，统计该窗口下的字母出现次数，记录最大次数为`maxCnt`，若`maxCnt+k<right-left+1`，即该窗口无法由同一个字母填满，那么`left++`。

```java
class Solution {
    public int characterReplacement(String s, int k) {
        if(s==null||s.length()==0) return 0;
        int left, right;
        left=right=0;
        char[] str=s.toCharArray();
        //统计字母出现频率
        int[] cnt=new int[26];
        int maxCnt=0;
        int maxIndex=0;
        int ans=0;
        while(left<=right&&right<str.length){
            int id=str[right]-'A';
            cnt[id]++;
            if(cnt[id]>maxCnt){
                maxIndex=id;
                maxCnt=cnt[id];
            }
            if(maxCnt+k<right-left+1){
                ans=Math.max(ans, maxCnt+k);
                cnt[str[left]-'A']--;
                left++;
                maxIndex=maxCnt=0;
                //重新计算出现频率和最高频率
                for(int i=0; i<26; i++){
                    if(cnt[i]>maxCnt){
                        maxCnt=cnt[i];
                        maxIndex=i;
                    }
                }
            }
            right++;
        }
        //注意别忘了
        ans=Math.max(ans, right-left);
        return ans;
    }
}
```

