# Repeated String Match

### 问题

字符串`a`，`b`。判断`a`是否能通过重复叠加的操作使得`b`是`k*a`的子串。求`k`的最小值。

注意`k=0`是，表示空值。

### 解决方案：时间复杂度$O(n^2)$

不知道`String.contains()`的实现是什么。如果是线性的，那么算法的时间复杂度应该是$O(n)$。

首先把`a`叠加至长度比`b`的更长些。那么`a`最多再叠加一次，如果还是不包含，那么再叠加多少次也没用。


```java
class Solution {
    public int repeatedStringMatch(String a, String b) {
        if(b==null||b=="") return 0;
        if(a.contains(b)) return 1;
        //注意正数向下取整，所以后面还要多考虑一次叠加
        int n=b.length()/a.length();
        StringBuilder sb=new StringBuilder();
        for(int i=0; i<n; i++){
            sb.append(a);
        }
        if(sb.toString().contains(b)) return n;
        sb.append(a);
        if(sb.toString().contains(b)) return n+1;
        sb.append(a);
        if(sb.toString().contains(b)) return n+2;
        else return -1;
    }
}
```

