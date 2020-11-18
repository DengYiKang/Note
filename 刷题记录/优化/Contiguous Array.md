# Contiguous Array

### 问题

有一个表示01字符串的整型数组`nums[1...n]`，求拥有相同0,1个数的连续子序列的最大长度。

### 解决方案

将0置为-1，那么原问题就变成求和为0的连续子序列的最大长度。

```java
class Solution {
    public int findMaxLength(int[] nums) {
        for(int i=0; i<nums.length; i++){
            if(nums[i]==0) nums[i]=-1;
        }
        int sum=0;
        HashMap<Integer, Integer> mp=new HashMap<>();
        mp.put(0, -1);
        int maxLen=0;
        for(int i=0; i<nums.length; i++){
            sum+=nums[i];
            int pre=mp.getOrDefault(sum, -2);
            if(pre!=-2) maxLen=Math.max(i-pre, maxLen);
            mp.putIfAbsent(sum, i);
        }
        return maxLen;
    }
}
```

