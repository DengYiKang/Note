# Card Flipping Game

### 问题

桌上有`n`张卡牌，两面印有数字，你可以将任意的牌翻面，然后取走某张牌的背面的数字，要求这个数字不出现在桌上卡牌的正面，求这个数的最小值。

### 解决方案：时间复杂度$O(n)$

如果你要取某个数字`k`，那么你可以把所有正面为`k`的牌翻转，除非它两面都是`k`。

```java
class Solution {
    public int flipgame(int[] fronts, int[] backs) {
        //记录两面都是相同值的数
        HashSet<Integer> same=new HashSet<>();
        for(int i=0; i<fronts.length; i++){
            if(fronts[i]==backs[i]) same.add(fronts[i]);
        }
        int ans=Integer.MAX_VALUE;
        for(int i=0; i<fronts.length; i++){
            if(!same.contains(fronts[i])) ans=Math.min(ans, fronts[i]);
        }
        for(int i=0; i<backs.length; i++){
            if(!same.contains(backs[i])) ans=Math.min(ans, backs[i]);
        }
        return ans==Integer.MAX_VALUE?0:ans;
    }
}
```

