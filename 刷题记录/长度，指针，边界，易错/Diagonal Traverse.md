# Diagonal Traverse

### 问题

蛇形打印一个二维数组的所有元素。

**Example:**

```
Input:
[
 [ 1, 2, 3 ],
 [ 4, 5, 6 ],
 [ 7, 8, 9 ]
]

Output:  [1,2,4,7,5,3,6,8,9]

Explanation:
```

<img src="../pic/7.png" style="zoom:50%;" />

### 解决方案

主要是找到方向与位置变换之间的关系。

设`dir[2][2]={{-1,1},{1,-1}}`。

当`d=0`时，优先向右转移，若非法，则向下转移，同时调整`d`。

当`d=1`时，优先向下转移，若非法，则向右转移，同时调整`d`。

```java
public class Solution {
    public int[] findDiagonalOrder(int[][] matrix) {
        if (matrix == null || matrix.length == 0) return new int[0];
        int m = matrix.length, n = matrix[0].length;
        
        int[] result = new int[m * n];
        int x = 0, y = 0, d = 0;
        int[][] dirs = {{-1, 1}, {1, -1}};
        
        for (int i = 0; i < m * n; i++) {
            result[i] = matrix[x][y];
            x += dirs[d][0];
            y += dirs[d][1];
            if(!isValid(x, y, matrix)){
                x-=dirs[d][0];
                y-=dirs[d][1];
                if(d==0){
                    if(isValid(x, y+1, matrix)) y+=1;
                    else x+=1;
                }else{
                    if(isValid(x+1, y, matrix)) x+=1;
                    else y+=1;
                }
                d=1-d;
            }
        }
        return result;
    }
    boolean isValid(int x, int y, int[][] matrix){
        if(x<0||x>=matrix.length) return false;
        if(y<0||y>=matrix[0].length) return false;
        return true;
    }
}
```

