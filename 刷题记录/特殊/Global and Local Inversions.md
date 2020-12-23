# Global and Local Inversions

### 问题

`global inversions`表示元组`(i, j)`的数量，使得`0<=i<j<N and A[i]>j`。

`local inversions`表示`i`的数量，使得`0<=i<N and A[i]>A[i+1]`。

判断以上两种数量是否相等。

### 解决方案：时间复杂度$O(n)$

`local`的表述可以写成`(i, i+1)`使得`0<=i<i+1<N and A[i]>A[i+1]`。那么`local`是`global`的子集。因此不能出现`(i,j)`使得`0<=i+1<j<N and A[i]>A[j]`的情况。

因为要判断是否相邻，所以保存两个索引`first_pos, second_pos`，有`first_pos<second_pos`。它们均保存遍历过程中的最大的两个数。对于当前的数，分情况判断即可。

```java
class Solution {
    public boolean isIdealPermutation(int[] A) {
        if(A==null||A.length<3) return true;
        int first_pos=0, second_pos=1;
        for(int i=2; i<A.length; i++){
            if(A[i]<A[first_pos]) return false;
            else if(A[i]<A[second_pos]&&second_pos!=i-1) return false;
            else{
                if(A[i]>Math.min(A[first_pos], A[second_pos])){
                    first_pos=A[first_pos]>A[second_pos]?first_pos:second_pos;
                    second_pos=i;
                }
            }
        }
        return true;
    }
}
```

