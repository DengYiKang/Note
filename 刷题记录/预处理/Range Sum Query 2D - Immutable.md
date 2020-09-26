# Range Sum Query 2D - Immutable

### 问题

输入一个二位矩阵`matrix`，两个坐标`(row1, col1),(row2, col2)`，分别为左上角坐标与右下角坐标，求出对应范围的和，有多组query

### 解决方案，$O(n)$

对原矩阵做预处理，$A[i][j]=\Sigma_{k=0}^{j}A[i][k]$，那么对给定的坐标就有：
$$
\sum_{k=row1}^{row2}(A[k][col2]-A[k][col1-1])
$$

```java
class NumMatrix {

    int[][] A;
    
    public NumMatrix(int[][] matrix) {
        if(matrix==null||matrix.length==0||matrix[0].length==0){
            A=null;
        }else{
            A=new int[matrix.length][matrix[0].length];
            for(int i=0; i<matrix.length; i++){
                for(int j=0; j<matrix[0].length; j++){
                    if(j==0) A[i][j]=matrix[i][j];
                    else A[i][j]=A[i][j-1]+matrix[i][j];
                }
            }
        }
    }
    
    public int sumRegion(int row1, int col1, int row2, int col2) {
        if(A==null) return 0;
        int ans=0;
        for(int i=row1; i<=row2; i++){
            if(col1>0) ans+=A[i][col2]-A[i][col1-1];
            else ans+=A[i][col2];
        }
        return ans;
    }
}
```

