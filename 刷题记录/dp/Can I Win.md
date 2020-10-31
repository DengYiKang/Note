# Can I Win

### 问题

现有`1~n`的数，两名选手先后挑选其中某个数，计算已获取的数字的累计和，每个数字只有一份。当某位选手选取数字后使得总和超过`target`时，这名选手获胜。对于给定的`n`与`target`，问第一名选手是否能获胜。

### 解决方案

#### 状态压缩dp，时间复杂度$O(2^n)$，空间复杂度$O(2^n)$

有以下状态能决定问题的解：

+ 待选的数字
+ 当前累积和

注意到第一个状态能推出第二个状态，因此最终只需考虑第一个状态。

定义`dp[status]`，表示状态为`status`时，**当前选手获胜的概率**。那么原问题的解可以转换为`dp[status0]`。

假设当前状态为`status`，当轮到选手A时，若能在这次选择中获胜，则返回`true`，**否则返回`!dp[next_status]`**。

```java
class Solution {
    HashMap<Integer, Boolean> mp;
    int n;
    int total;
    public boolean canIWin(int maxChoosableInteger, int desiredTotal) {
        mp=new HashMap<>();
        n=maxChoosableInteger;
        total=desiredTotal;
        if(total<=0) return true;
        if((n+1)*n/2<total) return false;
        return helper(0);
    }
    boolean helper(int status){
        if(mp.containsKey(status)) return mp.get(status);
        int rest=total;
        for(int i=1; i<=n; i++){
            if((status&(1<<i))!=0) rest-=i;
        }
        if(rest<=0) return false;
        //每一个可能操作若导致获胜，那么获胜
        for(int i=1; i<=n; i++){
            if((status&(1<<i))!=0) continue;
            int next_status=status|(1<<i);
            rest-=i;
            //!helper(next_status)表示对方失败
            if(rest<=0||!helper(next_status)){
                mp.put(status, true);
                return true;
            }
            rest+=i;
        }
        mp.put(status, false);
        return false;
    }
}
```

