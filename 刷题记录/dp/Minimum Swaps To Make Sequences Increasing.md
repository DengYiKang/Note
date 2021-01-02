## Minimum Swaps To Make Sequences Increasing

### 问题

`A`与`B`是等长的数字串，每次可以交换`A[i]`与`B[i]`，经过若干次交换后最终使得`A`与`B`是严格升序的。问交换的最小次数是多少？

### 解决方案：dp，时间复杂度$O(n)$

`dp_swap[i]`表示对于`A[1...i]`与`B[1...i]`而言，若交换末位的最小次数。

`dp_no_swap[i]`表示对于`A[1...i]`与`B[1...i]`而言，若不交换末位的最小次数。

```java
class Solution {
    public int minSwap(int[] A, int[] B) {
        int N=A.length;
        int[] dp_swap=new int[N];
        int[] dp_no_swap=new int[N];
        dp_no_swap[0]=0;
        dp_swap[0]=1;
        for(int i=1; i<N; i++){
            dp_no_swap[i]=dp_swap[i]=Integer.MAX_VALUE;
            if(A[i]>A[i-1]&&B[i]>B[i-1]){
                dp_no_swap[i]=Math.min(dp_no_swap[i], dp_no_swap[i-1]);
                dp_swap[i]=Math.min(dp_swap[i], dp_swap[i-1]+1);
            }
            if(A[i]>B[i-1]&&B[i]>A[i-1]){
                dp_no_swap[i]=Math.min(dp_no_swap[i], dp_swap[i-1]);
                dp_swap[i]=Math.min(dp_swap[i], dp_no_swap[i-1]+1);
            }
        }
        return Math.min(dp_swap[N-1],dp_no_swap[N-1]);
    }
}
```

