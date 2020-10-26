# Find All Duplicates in an Array

### 问题

现有一个数组`nums`，有`1<=nums[i]<=n`，其中`n`为`nums`的长度，其中的某些元素出现两次，其余的仅出现一次。求这些出现两次的元素。要求空间复杂度为$O(1)$。

### 解决方案：时间复杂度$O(n)$，空间复杂度$O(1)$

因为有`1<=nums[i]<=n`，因此可以把`nums[i]-1`映射到数组`nums`的元素里。

对于`index=nums[i]-1`，`nums[index]`的正负表示`nums[i]`出现次数为奇数还是偶数。而可以用`abs(nums[index])`来表示`nums[index]`本身代表的数。

```java
class Solution {
    public List<Integer> findDuplicates(int[] nums) {
        List<Integer> ans=new ArrayList<>();
        for(int i=0; i<nums.length; i++){
            int index=Math.abs(nums[i])-1;
            if(nums[index]<0) ans.add(Math.abs(nums[i]));
            else nums[index]=-nums[index];
        }
        return ans;
    }
}
```

