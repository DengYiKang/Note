# Score After Flipping Matrix

### 问题

二维01矩阵`A`，你可以将任意行和列翻转（即`0->1`或`1->0`）。最后将矩阵的所有行视为二进制数，求这些行的和数最大值。

### 解决方案：贪心，时间复杂度$O(n)$

每一行的第一个数如果为0，则对这行翻转。最终使得每一行的第一个数均为1.

从第二列开始，如果翻转该列能得到更多的1，则翻转。

```java
class Solution {
    public int matrixScore(int[][] A) {
        int N=A.length, M=A[0].length;
        for(int i=0; i<N; i++){
            if(A[i][0]==0) flipRow(A, i);
        }
        for(int i=1; i<M; i++){
            int columnSum=0;
            for(int j=0; j<N; j++){
                columnSum+=A[j][i];
            }
            if(columnSum*2<N) flipColumn(A, i);
        }
        int ans=0;
        for(int i=0; i<N; i++){
            int val=0;
            for(int j=0; j<M; j++){
                val<<=1;
                val+=A[i][j];
            }
            ans+=val;
        }
        return ans;
    }
    void flipRow(int[][] A, int row){
        for(int i=0; i<A[0].length; i++){
            A[row][i]^=1;
        }
    }
    void flipColumn(int[][] A, int column){
        for(int i=0; i<A.length; i++){
            A[i][column]^=1;
        }
    }
}
```

