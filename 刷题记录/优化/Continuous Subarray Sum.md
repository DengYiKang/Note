# Continuous Subarray Sum

### 问题

现有一个数组`nums[1...n]`与一个数`k`，问是否存在长度至少为2的连续子序列，使得其和为`k`的整数倍。

### 解决方案：时间复杂度$O(n)$

生成一个`sum[1...n]`的数组，对其做模`k`处理，那么如果`sum[i]-sum[j]=0`即`sum[i]=sum[j]`，其中`i>j+1`那么有解。

一开始只是用`HashSet`来存`sum`值，然后`i>j+1`这个条件不好判断。

后来用`HashMap`来存`sum`与`index`值，注意`index`值必须存放最旧的值。

```java
class Solution {
    public boolean checkSubarraySum(int[] nums, int k) {
        if(nums==null||nums.length==0) return false;
        int sum=0;
        HashMap<Integer, Integer> mp=new HashMap<>();
        mp.put(0, -1);
        for(int i=0; i<nums.length; i++){
            sum+=nums[i];
            if(k!=0) sum%=k;
            int pre=mp.getOrDefault(sum, -2);
            if(pre!=-2&&i-pre>1) return true;
            mp.putIfAbsent(sum, i);
        }
        return false;
    }
}
```



