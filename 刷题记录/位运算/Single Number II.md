# Single Number II

### 问题

输入一个非空的数组，其中一个元素只出现一次，其他元素出现三次，求出现一次的元素，要求时间复杂度为$O(n)$

### 解决方案

用整数类型变量ans表示该元素的值。对某一bit进行计数，所有元素在该bit为1的总次数n%3即为解在该bit上的值。

```java
class Solution {
    public int singleNumber(int[] nums) {
        int ans=0;
        for(int i=0; i<32; i++){
            int sum=0;
            for(int j=0; j<nums.length; j++){
                if(((nums[j]>>i)&1)==1) sum++;
            }
            sum%=3;
            ans|=sum<<i;
        }
        return ans;
    }
}
```

一个元素m次，其他元素n次，其中m<n，均可使用这种方式