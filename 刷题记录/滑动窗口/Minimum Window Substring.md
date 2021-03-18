# Minimum Window Substring

### 问题

两个字符串`s`和`t`，要求找出`s`中的最小连续子串，使得该子串包含`t`中所有的字符。

### 解决方案：滑动窗口，时间复杂度$O(n)$

lo=hi=0。每次向右移动hi，维护当前窗口的包含各个字符的数量，同时引入`need`用来确定当前窗口是否包含所有的`t`中的字符（还没加入hi之前，`s[hi]`的数量小于需要的数量，那么need减一，否则不增不减）。同时，当窗口包含所有`t`中的字符时，以后将一直保持该性质。如果hi加入后使得`s[hi]`这个字符超过了所需，那么检查`s[lo]`是否超过，如果超过那么不断更新lo，直到`s[lo]`恰好满足所需。

```java
class Solution {
        public String minWindow(String s, String t) {
                HashMap<Integer, int[]> loc=new HashMap<>();
                HashMap<Character, Integer> t_cnt=new HashMap<>();
                HashMap<Character, Integer> s_cnt=new HashMap<>();
                int lo=0, need=t.length(), min_width=Integer.MAX_VALUE;
                for(int i=0; i<t.length(); i++){
                        char ch=t.charAt(i);
                        t_cnt.put(ch, t_cnt.getOrDefault(ch, 0)+1);
                }
                for(int hi=0; hi<s.length(); hi++){
                        char ch=s.charAt(hi);
                        if(t_cnt.getOrDefault(ch, 0)<=s_cnt.getOrDefault(ch, 0)){
                                s_cnt.put(ch, s_cnt.getOrDefault(ch, 0)+1);
                                while(lo<=hi
                                      &&s_cnt.get(s.charAt(lo))>t_cnt.getOrDefault(s.charAt(lo), 0)){
                                        int tmp=s_cnt.get(s.charAt(lo));
                                        s_cnt.put(s.charAt(lo), tmp-1);
                                        lo++;
                                }
                        }else{
                                s_cnt.put(ch, s_cnt.getOrDefault(ch, 0)+1);
                                need--;
                        }
                        if(need==0&&hi-lo<min_width){
                                loc.put(hi-lo, new int[]{lo, hi});
                                min_width=hi-lo;
                        }
                }
                int[] ans=loc.getOrDefault(min_width, new int[]{-1,-1});
                return ans[0]>=0?s.substring(ans[0],ans[1]+1):"";
        }
        
}
```

