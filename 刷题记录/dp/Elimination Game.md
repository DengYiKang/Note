# Elimination Game

### 问题

现有`1～n`的数组，对它进行这样的处理：从左到右把奇数位的数去掉，再从右到左把奇数位的数去掉。不断反复，直到最后只剩一个数，那么输出这个数。

### 解决方案：dp，时间复杂度$O(logn)$，空间复杂度$O(logn)$

考虑当n为奇数时，在第一轮会被消去，那么第一轮消去后的结果跟n-1相同。

当n为偶数时，考虑经过前两轮后的情况，可以计算此时的剩余数的长度为`n/4`，若`(n/2)%2=1`，那么开头是`4`；若`(n/2)%2=0`，那么开头是`2`，并且它们成等差数列。

注意到，原数组是`1～n`的排列，那么意味着`dp[n]`也可以表示为n个数进行相同的操作后剩余的数的索引。因此这里可以应用到前两轮后的结果，直接根据剩余数的索引求得结果。

```java
class Solution {
    public int lastRemaining(int n) {
        if(n<1) return 0;
        if(n==1) return 1;
        else if(n==2) return 2;
        else if(n==3) return 2;
        int[] dp=new int[n+1];
        dp[1]=1;
        dp[2]=2;
        dp[3]=2;
        for(int i=4; i<n+1; i++){
            if(i%2==1) dp[i]=dp[i-1];
            else{
                int len=i/4;
                int offset=0;
                if((i/2)%2==1){
                    offset=0;
                }else{
                    offset=-2;
                }
                offset+=4*dp[len];
                dp[i]=offset;
            }
        }
        return dp[n];
    }
}
```

然而上述代码`[Memory Limit Exceeded]`，之前的某些dp状态不需要保存，因此可以改成递归形式：

```java
class Solution {
    public int lastRemaining(int n) {
        if(n<1) return 0;
        return getDp(n);
    }
    int getDp(int n){
        if(n==1) return 1;
        else if(n==2) return 2;
        else if(n==3) return 2;
        if(n%2==1) return getDp(n-1);
        int offset=(n/2)%2==1?0:-2;
        offset+=4*getDp(n/4);
        return offset;
    }
}
```

