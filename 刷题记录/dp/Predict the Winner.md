# Predict the Winner

### 问题

有两名玩家在玩一个游戏，一个玩家每轮可以从池子里拿走一个数，使得自己的总数增大。当池子里的数全部被拿完之后，哪个玩家的数大，哪个玩家赢。如果相等，那么第一个玩家赢。

### 解决方案

与Can I Win的问题类似，但是那种方法放在这不太适合，因为有“如果相等，那么第一个玩家赢”的条件，在递归函数中判断这种情况时比较难处理。因此有必要把判断放在递归函数外处理。

一般地，如果要确定某个状态，第一个玩家是否能赢，那么有以下状态决定：

+ left
+ right
+ 该玩家当前sum

考虑使用差的形式，`memo[left][right]`表示初始状态下数的可选范围为`[left~right]`时，到最后第一个玩家对第二个的结果差是多少。那么就可以把判断放在递归函数外处理了。

```java
public class Solution {
    public boolean PredictTheWinner(int[] nums) {
        Integer[][] memo = new Integer[nums.length][nums.length];
        return winner(nums, 0, nums.length - 1, memo) >= 0;
    }
    public int winner(int[] nums, int s, int e, Integer[][] memo) {
        if (s == e)
            return nums[s];
        if (memo[s][e] != null)
            return memo[s][e];
        int a = nums[s] - winner(nums, s + 1, e, memo);
        int b = nums[e] - winner(nums, s, e - 1, memo);
        memo[s][e] = Math.max(a, b);
        return memo[s][e];
    }
}
```

