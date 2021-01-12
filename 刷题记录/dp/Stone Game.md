# Stone Game

### 问题

`piles[1...n]`表示有n个数，这n个数排成一排。两个人轮流取最左边或者最右边的数，为了使最终手里的总数尽可能地大，他们采取最优的策略，最终谁的总数最大谁获胜。假设总数为奇数，且n为偶数。问第一个人是否能获胜。

### 解决方案：

#### 一、递归，时间复杂度$O(2^n)$，TLE

```java
class Solution {
    HashMap<String, Boolean> memo=new HashMap<>();
    public boolean stoneGame(int[] piles) {
        int tot=0;
        memo.clear();
        for(int pile:piles){
            tot+=pile;
        }
        return canwin(piles, 0, piles.length-1, 0, tot);
    }
    public boolean canwin(int[] piles, int l, int r, int sum, int tot){
        String code=l+","+r+","+sum;
        if(memo.containsKey(code)) return memo.get(code);
        if(l==r){
            sum+=piles[l];
            return sum*2>tot;
        }
        int otherSum=tot-sum;
        for(int i=l; i<=r; i++){
            otherSum-=piles[i];
        }
        boolean res=!canwin(piles, l+1, r, otherSum, tot)
            || !canwin(piles, l, r-1, otherSum, tot);
        memo.put(code, res);
        return res;
    }
}
```

#### 二、dp，时间复杂度$O(n^2)$

用`dp[i][j]`表示开局为`piles[i], piles[i+1],...,piles[j-1],pile[j]`时，第一个人比第二个人多的分数。

如果此时开局为奇数个，那么应该由第二个人先取，否则由第一个人先取。

```java
class Solution {
    public boolean stoneGame(int[] piles) {
        int N=piles.length;
        //留出0和N来防止越界
        int[][] dp=new int[N+2][N+2];
        for(int i=N; i>=1; i--){
            for(int j=i; j<=N; j++){
                if((j-i+1)%2==1){
                    dp[i][j]=Math.min(dp[i+1][j]-piles[i-1], dp[i][j-1]-piles[j-1]);
                }else{
                    dp[i][j]=Math.max(dp[i+1][j]+piles[i-1], dp[i][j-1]+piles[j-1]);
                }
            }
        }
        return dp[1][N]>0;
    }
}
```

