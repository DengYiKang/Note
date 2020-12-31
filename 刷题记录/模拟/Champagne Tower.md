# Champagne Tower

### 问题

倒`poured`量的酒，每个杯子最多容纳1量的酒，求给定行数`query_row`和对应行的第`query_glass`个杯子的酒的量。

<img src="../pic/14.png" style="zoom:50%;" />

### 解决方案：模拟，时间复杂度$O(n)$

对每个结点，只需要模拟进入的流量（或输出的流量），保存的流量可以通过

`Math.min(1, 进入的流量)`来求得。

```java
class Solution {
    public double champagneTower(int poured, int query_row, int query_glass) {
        double[][] A=new double[102][102];
        A[0][0]=(double) poured;
        for(int r=0; r<query_row; r++){
            for(int c=0; c<=r; c++){
                double q=(A[r][c]-1.0)/2.0;
                if(q>0){
                    A[r+1][c]+=q;
                    A[r+1][c+1]+=q;
                }
            }
        }
        return Math.min(1, A[query_row][query_glass]);
    }
}
```

