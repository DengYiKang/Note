# Flip String to Monotone Increasing

### 问题

字符串`S`，由`0,1`组成。将`S`中的任意个`0`或`1`翻成`1`或`0`，使得所有的1在所有的0的右边。求最少的翻转次数。

### 解决方案：时间复杂度$O(n)$

即搜索索引`i`，`i`为连续的`0,1`交界处。

设`p[i]`表示左边`i`个数的`1`的个数，即`p[i]=A[0]+A[1]+A[2]+...+A[i]`。

如果使左边`i`个数为0，右边剩余的数为1，那么翻转次数为`p[i]+N-i-(p[N]-p[i])`。

```java
class Solution {
    public int minFlipsMonoIncr(String S) {
        int[] p=new int[S.length()+1];
        for(int i=0; i<S.length(); i++){
            if(S.charAt(i)=='1') p[i+1]=p[i]+1;
            else p[i+1]=p[i];
        }
        int ans=Integer.MAX_VALUE;
        for(int i=0; i<S.length()+1; i++){
            int tmp=p[i]+(S.length()-i-(p[S.length()]-p[i]));
            ans=Math.min(ans, tmp);
        }
        return ans;
    }
}
```

