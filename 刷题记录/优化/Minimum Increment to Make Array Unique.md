# Minimum Increment to Make Array Unique

### 问题

给一组数，一步定义为将某个元素加一，求将这组数的所有的元素各不相同的最小步数。

### 解决方案：时间复杂度$O(n)$ 

假设有多种不同的变换方法，最终结果（即最终数组的元素，不考虑顺序）只有一个。

设原数组为A，最小步数对应的任意的结果数组为B，那么sum(B)-sum(A)只有一个固定的值。

因此增1的顺序无关紧要。

先将A进行排序，遍历A时，需要搜索比A[i]大的还未使用过的最小数。但是这种情况下的时间复杂度的上界为$O(n^2)$。

注意到，将A进行排序后，结果数组B为单调递增的数列。因此可以从结果来考虑。

定义变量`apply`作为被取的结果数值，被取时更新。

```java
class Solution {
    public int minIncrementForUnique(int[] A) {
            int apply=0;
            int sum1=0, sum2=0;
            Arrays.sort(A);
            for(int x:A){
                    sum1+=x;
            }
            for(int i=0; i<A.length; i++){
                    int j=A[i];
                    if(apply<j) apply=j;
                    sum2+=apply;
                    apply++;
            }
            return sum2-sum1;
    }
}
```

