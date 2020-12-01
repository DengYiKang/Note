# Out of Boundary Paths

### 问题

初始点`(x,y)`，在`m*n`的块中，有步数限制`N`，问有多少种走法出块。

### 解决方案：记忆化dfs

如果让dfs具有返回值，那么其本身就是一个dp函，就可以用记忆化dp的思想来优化。

```java
class Solution {
    int[][] step={{1,0},{-1,0},{0,1},{0,-1}};
    HashMap<Integer, Integer> mp=new HashMap<>();
    public int findPaths(int m, int n, int N, int i, int j) {
        mp.clear();
        return dfs(m, n, i, j, N);
    }
    public int dfs(int m, int n, int x, int y, int need){
        int code=getHashCode(x, y, need);
        if(mp.containsKey(code)) return mp.get(code);
        if(isOut(m, n, x, y)){
            return 1;
        }
        if(need==0) return 0;
        if(!isOut(m, n, x-need, y)&&!isOut(m, n, x+need, y)&&
          !isOut(m, n, x, y+need)&&!isOut(m, n, x, y-need)){
            mp.put(code, 0);
            return 0;
        }
        int cnt=0;
        for(int i=0; i<4; i++){
            int nx=x+step[i][0];
            int ny=y+step[i][1];
            cnt+=dfs(m, n, nx, ny, need-1);
            cnt%=(int)(Math.pow(10, 9)+7);
        }
        mp.put(code, cnt);
        return cnt;
    }
    public boolean isOut(int m, int n, int x, int y){
        if(x<0||x>=m) return true;
        if(y<0||y>=n) return true;
        return false;
    }
    public int getHashCode(int x, int y, int need){
        return x*51*51+y*51+need;
    }
}
```

