# Total Hamming Distance

### 问题

现有一个整型数组，要求计算该数组的所有元素之间的海明距离之和，海明距离定义为两数bit位上不同的个数。

### 解决方案，时间复杂度$O(n)$，空间复杂度$O(1)$

如果每个数间进行比较的话，那么时间复杂度为$O(n^2)$。

可以从bit位上考虑，如果某位为1的数的个数为n，为0的数为m，那么对于这个位的海明距离和就为n*m。

```java
class Solution {
    public int totalHammingDistance(int[] nums) {
        if(nums==null||nums.length==0) return 0;
        int ans=0;
        for(int bit=0; bit<32; bit++){
            int count=0;
            for(int i=0; i<nums.length; i++){
                if(((nums[i]>>bit)&1)==1) count++;
            }
            ans+=count*(nums.length-count);
        }
        return ans;
    }
}
```

