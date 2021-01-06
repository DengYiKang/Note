# Split Array into Fibonacci Sequence

### 问题

将字符串`S`分割成斐波那契数列，要求数列个数大于等于3，且每个数小于$2^{31}-1$。

### 解决方案：时间复杂度$O(81n)$

只要前两个确定了，后面的数都确定了。

比较简单，主要考察熟练程度。

```java
class Solution {
    public List<Integer> splitIntoFibonacci(String S) {
        long p=0, q=0;
        for(int i=1; i<=10&&i<S.length(); i++){
            if(S.charAt(0)=='0'&&i>1) continue;
            p=Long.valueOf(S.substring(0, i));
            if(p>Integer.MAX_VALUE) continue;
            for(int j=i+1; j<=i+10&&j<S.length(); j++){
                if(S.charAt(i)=='0'&&j>i+1) continue;
                q=Long.valueOf(S.substring(i, j));
                if(q>Integer.MAX_VALUE) continue;
                int k=j;
                long need=p+q;
                long x=p, y=q;
                List<Integer> res=new ArrayList<>();
                res.add((int) p);
                res.add((int) q);
                while(k!=S.length()){
                    if(need>Integer.MAX_VALUE) break;
                    String need_str=String.valueOf((int) need);
                    if(!S.substring(k).startsWith(need_str)) break;
                    res.add((int) need);
                    x=y;
                    y=need;
                    need=x+y;
                    k+=need_str.length();
                }
                if(k==S.length()) return res;
            }
        }
        return new ArrayList<>();
    }
}
```

