# Jump Game

### 问题

输入一个数组A[1...n]，A[i]表示第i个元素能向前走的最大步数，问从第一个元素开始，能否走到最后一个元素？

### 解决方案

之前考虑复杂了，用并查集和dfs做的都是几百ms。

为什么之前使用并查集和dfs做的呢，因为并没有注意到从0端出发可走的所有合法位置都是连续的，因此只需要一次遍历即可。

注意该题的特征，从左到右扫描，扫描到i点，其跳转范围是[i+1, i+A[i]]，这些范围内的点都是合法的点，把这些点加入到搜索范围，直到搜索范围内出现最后一个元素。

```java
class Solution {
    
    public boolean canJump(int[] A) {
        if(A.length==1) return true;
        int lo=0, hi=A[0];
        while(lo<=hi){
            hi=Math.max(hi, lo+A[lo]);
            if(hi>=A.length-1) return true;
            lo++;
        }
        return false;
    }
    
}
```

