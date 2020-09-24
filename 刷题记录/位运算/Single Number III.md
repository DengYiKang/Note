# Single Number III

### 问题

输入一个数组`nums`，其中两个元素只出现一次，其他元素只出现两次，求这两个元素

### 解决方案：时间复杂度$O(n)$，空间复杂度$O(1)$ 

+ `diff = nums[0] xor ... xor nums[n] = a xor b`，其中`a,b`是问题的解
+ `diff&=-diff`，求出`diff`中为1的最低位（补码就是最低位的1不变，其它高位取反）
+ `diff`中为1的位表示`a`与`b`在该位上不同
+ 采用`nums[i]&diff==0`来分组，`a`与`b`必然在不同组，且每个组除`a,b`外必然是复数个
+ 每个组的所有元素异或，即可得到解

```java
class Solution {
    public int[] singleNumber(int[] nums) {
        int diff=0;
        for(int num:nums){
            diff^=num;
        }
        diff&=-diff;
        int[] ans={0,0};
        for(int num:nums){
            if((num&diff)==0){
                ans[0]^=num;
            }else{
                ans[1]^=num;
            }
        }
        return ans;
    }
}
```

