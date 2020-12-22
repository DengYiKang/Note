# Largest Plus Sign

### 问题

有个01矩阵，求1所构成的`+`号的最大半径长，`+`号是中心对称的。

### 解决方案：dp，时间复杂度$O(n^2)$

对于一个坐标`(i, j)`，需要计算它的四方的连续`1`的长度，它们的最小值就是半径。

那么可以用dp的方式分别计算出`l[i][j], r[i][j], t[i][j], d[i][j]`，最后再遍历求解：

```java
class Solution {
    public int orderOfLargestPlusSign(int N, int[][] mines) {
        int[][] l=new int[N+2][N+2];
        int[][] r=new int[N+2][N+2];
        int[][] t=new int[N+2][N+2];
        int[][] d=new int[N+2][N+2];
        HashSet<Integer> mine_set=new HashSet<>();
        for(int[] mine:mines){
            mine_set.add(hashCode(mine[0]+1, mine[1]+1));
        }
        for(int i=0; i<N+2; i++){
            mine_set.add(hashCode(0, i));
            mine_set.add(hashCode(i, 0));
            mine_set.add(hashCode(N+1, i));
            mine_set.add(hashCode(i, N+1));
        }
        for(int i=1; i<N+1; i++){
            for(int j=1; j<N+1; j++){
                int code=hashCode(i, j);
                boolean isMine=mine_set.contains(code);
                l[i][j]=isMine?0:l[i][j-1]+1;
                t[i][j]=isMine?0:t[i-1][j]+1;
            }
        }
        for(int i=N; i>=1; i--){
            for(int j=N; j>=1; j--){
                int code=hashCode(i, j);
                boolean isMine=mine_set.contains(code);
                r[i][j]=isMine?0:r[i][j+1]+1;
                d[i][j]=isMine?0:d[i+1][j]+1;
            }
        }
        int ans=0;
        for(int i=1; i<N+1; i++){
            for(int j=1; j<N+1; j++){
                int order=Math.min(Math.min(l[i][j], r[i][j]),
                             Math.min(t[i][j], d[i][j]));
                ans=Math.max(ans, order);
            }
        }
        return ans;
    }
    public int hashCode(int x, int y){
        return x*505+y;
    }
    
}
```

注意，可以只用一个`dp`数组，对`l[i][j]`等数组的更新可以直接在`dp`数组上更新，可以减小空间。