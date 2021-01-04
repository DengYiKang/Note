# Rabbits in Forest

### 问题

森林里面有一群兔子，各自有着一种颜色，询问它们一个问题，有多少其他的兔子跟自己的颜色相同？现给出一部分兔子的回答，求森林里最少有多少只兔子。

### 解决方案：时间复杂度$O(n)$

这是一个匹配问题。回答不同数字的兔子肯定不是同一种颜色，因此只能是相同数字之间进行匹配。先对数组进行排序，尽可能地将回答同一个数字的分为同一种颜色，如果超出数量则将当前数字+1累加至结果，并且重新开始该数字的匹配。

```java
class Solution {
    public int numRabbits(int[] A) {
        if(A==null||A.length==0) return 0;
        Arrays.sort(A);
        int cnt=0, cur=-1, ans=0;
        for(int i=0; i<A.length; i++){
            if(cur==-1){
                cur=A[i];
                cnt=1;
            }else if(cur!=A[i]){
                ans+=cur+1;
                cur=A[i];
                cnt=1;
            }else{
                cnt++;
                if(cnt>cur+1){
                    //更新结果，重新匹配
                    ans+=cur+1;
                    cnt=1;
                }
            }
        }
        if(cnt!=0) ans+=cur+1;
        return ans;
    }
}
```

