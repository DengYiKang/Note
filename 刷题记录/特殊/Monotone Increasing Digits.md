# Monotone Increasing Digits

### 问题

`N`为一个非负的整数，求不比`N`大的整数`num`，要求`num`的数字为单调递增的（即`nums[i]<=nums[i+1]`）

### 解决方案

对于`nums`每位先取与`N`对应位相等的数，如果有`nums[i]>nums[i-1]`，那么找到第`j`位，使得`nums[j-1]<nums[j]=nums[j+1]=...=nums[i-1]<nums[i]`，将`nums[j]--`，然后对所有的大于`j`的`i`，把它们都赋值为9。

注意到，被改写的`nums[j]`不可能是0，因为若`nums[j]`为0，那么遍历到`j`位时必然已经进行赋值9的操作了。

```java
class Solution {
    public int monotoneIncreasingDigits(int N) {
        char[] target=String.valueOf(N).toCharArray();
        char[] nums=new char[target.length];
        for(int i=0; i<target.length; i++){
            nums[i]=target[i];
            if(i>0&&nums[i]<nums[i-1]){
                int p=i-1, val=nums[p];
                while(p-1>=0&&nums[p]==nums[p-1]) p--;
                nums[p]--;
                while(++p<nums.length) nums[p]=9+'0';
                break;
            }
        }
        int ans=0;
        for(int i=0; i<nums.length; i++){
            int num=nums[i]-'0';
            ans=ans*10+num;
        }
        return ans;
    }
}
```

