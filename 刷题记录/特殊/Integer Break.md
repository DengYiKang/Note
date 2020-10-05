# Integer Break

### 问题

现有一个数字`n`，对其进行分解，分解成`k`个数之和，这`k`个数的积为`p`，求`p`的最大值

### 解决方案：math，时间复杂度$O(n)$，空间复杂度$O(1)$

高中学过不等式就很容易想到了。

```java
class Solution {
    public int integerBreak(int n) {
        int ans=Integer.MIN_VALUE;
        for(int i=2; i<=n; i++){
            int c=n/i, remain=n%i, product=1;
            for(int j=0; j<i; j++){
                if(remain>0){
                    product*=c+1;
                    remain--;
                }else{
                    product*=c;
                }
            }
            ans=Math.max(ans, product);
        }
        return ans;
    }
}
```

