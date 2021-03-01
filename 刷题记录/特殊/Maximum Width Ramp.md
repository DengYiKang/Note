# Maximum Width Ramp

### 问题

整型数组`A`，一个ramp表示一个二元组`(i, j)`，`i < j and A[ i ]<=A[ j ] `，ramp的长度为`j - i`。求ramp的最大长度。

### 解决方案：时间复杂度$O(nlogn)$，空间复杂度$O(n)$ 

将A进行排序，值替换成相应的序号。转换为求`A[ j ]-A[ i ], j > i`的最大值。

在遍历时，可以维护A[i]最小值。

```java
class Solution {
        public int maxWidthRamp(int[] A) {
                int N=A.length;
                Integer[] B=new Integer[A.length];
                for(int i=0; i<N; i++){
                        B[i]=i;
                }
                Arrays.sort(B, (i, j)->(A[i]-A[j]));
                int max_val=0;
                int min_val=N;
                for(int x:B){
                        max_val=Math.max(x-min_val, max_val);
                        min_val=Math.min(x, min_val);
                }
                return Math.max(max_val, 0);
        }
}
```

