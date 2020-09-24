# Search a 2D Matrix II

### 问题

输入一个m*n矩阵，每行每列从小到大排列，要求判断一个数是否为该矩阵的元素

### 解决方案：时间复杂度$O(m+n)$，空间复杂度$O(1)$ 

从右上角开始搜索。**注意，只有从右上角的点出发，两个方向产生的结果是逆向的（即增减相反），这是一个重要特征。**

![0_1488858509025_Monosnap 2017-03-06 22-48-17.jpg](../pic/3.png)

```java
class Solution {
    
    public boolean searchMatrix(int[][] matrix, int target) {
        if(matrix==null||matrix.length==0||matrix[0].length==0) return false;
        int x=0, y=matrix[0].length-1;
        while(x<matrix.length&&y>=0){
            if(matrix[x][y]>target) y--;
            else if(matrix[x][y]<target)  x++;
            else return true;
        }
        return false;
    }
}
```

