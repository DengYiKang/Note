# Remove Duplicate Letters

### 问题

现有一个字符串`s`，移除所有重复的字母，并生成最小字典序的字符串，求该生成串

### 解决方案：时间复杂度：$O(n^2)$，空间复杂度：$O(n^2)$

```java
public class Solution {
    public String removeDuplicateLetters(String s) {
        int[] cnt = new int[26];
        int pos = 0; // 最小的字符的索引
        for (int i = 0; i < s.length(); i++) cnt[s.charAt(i) - 'a']++;
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) < s.charAt(pos)) pos = i;
            //保证生成串的顺序性
            if (--cnt[s.charAt(i) - 'a'] == 0) break;
        }
        return s.length() == 0 ? "" : s.charAt(pos) + removeDuplicateLetters(s.substring(pos + 1).replaceAll("" + s.charAt(pos), ""));
    }
}
```

