# Counting Bits

### 问题

求出`[0,n]`区间的所有整数的二进制表示中1的个数。例如：

**Example:**

```
Input: 5
Output: [0,1,1,2,1,2]
```

### 解决方案：时间复杂度$O(n)$，空间复杂度$O(n)$

```java
class Solution {
    public int[] countBits(int num) {   
        int[] dp=new int[num+1];
        dp[0]=0;
        for(int i=1; i<=num; i++){
            dp[i]=dp[i>>1]+i%2;
        }
        return dp;
    }
}
```

