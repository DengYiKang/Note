# Camelcase Matching

### 问题

给一个字符串pattern，向它任意位置插入任意的小写字母，得到query，那么pattern与query是匹配的。

给定queries[]和pattern，判断它们是否是匹配的。

### 解决方案：双指针，时间复杂度$O(nm)$

```java
class Solution {
        
        public List<Boolean> camelMatch(String[] queries, String pattern) {
                List<Boolean> ans=new ArrayList<>();
                for(String query:queries){
                        ans.add(doMatch(query.toCharArray(), pattern.toCharArray()));
                }
                return ans;
        }
        public boolean doMatch(char[] query, char[] pattern){
                int i=0, j=0;
                while(i<query.length&&j<pattern.length){
                        if(query[i]==pattern[j]&&++i>0&&++j>0) continue;
                        if(Character.isUpperCase(query[i])) return false;
                        i++;
                }
                if(j<pattern.length) return false;
                if(i<query.length){
                        while(i<query.length){
                                if(Character.isUpperCase(query[i])) return false;
                                i++;
                        }
                }
                return true;
        }
}
```

