# Knight Probability in Chessboard

### 问题

国际象棋里，`N*N`的棋盘上有一个骑士，可以走八个方向。现使该棋子走`K`步，求该棋子仍在棋盘上的概率。

### 解决方案：记忆化dfs，时间复杂度$O(N^2K)$

这个题目对概率的定位有点问题，我认为的概率是指`count(棋子最终在棋盘上的路线数)/count(棋子的总路线数)`。而在计算总路线数时，棋子若在棋盘外，则应该终止后续的行动（题目也是这样想的），但是题目简单的把棋子的总路线数计算为$8^K$。

以下的代码同时计算了我认为的棋子最终在棋盘上的路线数和棋子的总路线数，但计算答案时总路线数仍用$8^K$的式子。

```java
class Solution {
    int[][] step=new int[][]{{1,2}, {2,1}, {2,-1}, {1,-2}, {-1,-2}, {-2,-1}, {-2,1}, {-1,2}};
    HashMap<Integer, double[]> mp=new HashMap<>();
    public double knightProbability(int N, int K, int r, int c) {
        mp.clear();
        double[] ans=go(r, c, N, K);
        return ans[0]/(Math.pow(8, K));
    }
    public double[] go(int x, int y, int N, int need){
        if(!isValid(x, y, N)) return new double[]{0, 1};
        int code=hashCode(x, y, need);
        if(mp.containsKey(code)) return mp.get(code);
        if(need==0) return new double[]{1,1};
        double[] ans=new double[2];
        for(int i=0; i<8; i++){
            int nx=x+step[i][0];
            int ny=y+step[i][1];
            double[] tmp=go(nx, ny, N, need-1);
            ans[0]+=tmp[0];
            ans[1]+=tmp[1];
        }
        mp.put(code, ans);
        return ans;
    }
    public int hashCode(int x, int y, int need){
        int code=x;
        code=code*101+y;
        code=code*101+need;
        return code;
    }
    public boolean isValid(int x, int y, int N){
        if(x<0||x>=N) return false;
        if(y<0||y>=N) return false;
        return true;
    }
}
```

