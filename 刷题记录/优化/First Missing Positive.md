# First Missing Positive

### 问题

乱序的数组`nums`，找到最小的缺失的正数。

### 解决方案：时间复杂度$O(n)$，空间复杂度$O(1)$

要求时间复杂度$O(n)$，空间复杂度$O(1)$，自然能想到需要同时用`nums`作为标记数组。

注意到只需要找1～nums.length中哪个缺失的数，因此可以用`nums[i]`的值来表示i+1的缺失情况。

首先将数组中所有元素为非正数的数置为-1。用`nums[i]=0`来表示i+1这个值存在。

扫描到`nums[i]`时，需要将`nums[nums[i]-1]`置为0，如果`nums[nums[i]-1]`为正数，那么也要递归地设置。

```java
class Solution {
        public int firstMissingPositive(int[] nums) {
                for(int i=0; i<nums.length;i++){
                        if(nums[i]<=0) nums[i]=-1;
                }
                for(int i=0; i<nums.length; i++){
                        int target=nums[i]-1;
                        if(target>=nums.length||target<0) continue;
                        while(target<nums.length&&target>=0&&nums[target]!=0){
                                int tmp=nums[target];
                                nums[target]=0;
                                target=tmp-1;
                        }
                }
                for(int i=0; i<nums.length; i++){
                        if(nums[i]!=0) return i+1;
                }
                return nums.length+1;
        }
        
}
```

