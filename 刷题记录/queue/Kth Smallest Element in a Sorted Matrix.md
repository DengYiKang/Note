## Kth Smallest Element in a Sorted Matrix

### 问题

现有一个`n*n`矩阵，每行每列是升序的，找到第k小的元素。

### 解决方案：优先队列

同`Find K Pairs with Smallest Sums`题目。

```java
class Solution {
    public int kthSmallest(int[][] matrix, int k) {
        if(matrix==null||matrix.length==0||matrix[0].length==0) return 0;
        PriorityQueue<int[]> queue=new PriorityQueue<>(
            (a, b)->matrix[a[0]][a[1]]-matrix[b[0]][b[1]]
        );
        for(int i=0; i<matrix.length&&i<k; i++){
            queue.offer(new int[]{0, i});
        }
        int[] pair=null;
        for(int i=0; i<k; i++){
            pair=queue.poll();
            if(pair[0]==matrix.length-1) continue;
            queue.offer(new int[]{pair[0]+1, pair[1]});
        }
        return matrix[pair[0]][pair[1]];
    }
}
```

