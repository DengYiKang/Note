# Regular Expression Matching

### 问题

正则匹配问题，pattern由小写字母、`.`与`*`组成。给定text以及pattern，问是否能够匹配。

### 解决方案：dp，时间复杂度$O(mn)$

`match(lo, hi, s, p, dup)`，表示正在匹配`s[lo]`与`p[hi]`，`dup[hi]=true`表示`p[hi]`由`*`所修饰。

```java
class Solution {
        HashMap<Integer, Boolean> mp;
        public boolean isMatch(String s, String p) {
                StringBuilder sb=new StringBuilder();
                boolean[] dup=new boolean[p.length()];
                mp=new HashMap<>();
                for(char ch:p.toCharArray()){
                        if(ch=='*') dup[sb.length()-1]=true;
                        else sb.append(ch);
                }
                return match(0, 0, s, sb.toString(), dup);
        }
        public boolean match(int lo, int hi, String s, String p, boolean[] dup){
                int code=lo*100+hi;
                if(mp.containsKey(code)) return mp.get(code);
                if(lo>=s.length()){
                        if(hi>=p.length()) return true;
                        else if(dup[hi]) return match(lo, hi+1, s, p, dup);
                        else return false;
                }else if(hi>=p.length()) return false;
                // lo<s.length&&hi<p.length
                boolean ans=false;
                char sch=s.charAt(lo);
                char pch=p.charAt(hi);
                if(pch!='.'){
                        if(sch==pch){
                                ans=ans||match(lo+1, hi+1, s, p, dup);
                                if(dup[hi]) ans=ans||match(lo+1, hi, s, p, dup)||match(lo, hi+1, s, p, dup);
                        }else ans=ans||(dup[hi]?match(lo, hi+1, s, p, dup):false);
                }else{
                        if(!dup[hi]) ans=ans||match(lo+1, hi+1, s, p, dup);
                        else {
                                for(int len=0; lo+len<=s.length(); len++){
                                        ans=ans||match(lo+len, hi+1, s, p, dup);
                                        if(ans) break;
                                }
                        }
                }
                mp.put(code, ans);
                return ans;
        }
}
```

