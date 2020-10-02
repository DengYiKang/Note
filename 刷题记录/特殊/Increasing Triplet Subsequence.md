# Increasing Triplet Subsequence

### 问题

现有一个乱序的数组`A`，判断是否存在`i,j,k(i<j<k)`，使得`A[i]<A[j]<A[k]`

### 解决方案：时间复杂度$O(n)$，空间复杂度$O(n)$

设想一下，当你找到两数`a,b`后，你期待第三个数`c`，但现在接受到了比`a`更小的数`d`，那么`d`也有发展的潜力。因此这里保存一组数的最小值和两组数的最小值。

```java
class Solution {
    public boolean increasingTriplet(int[] nums) {
        if(nums==null||nums.length==0) return false;
        int max1=Integer.MIN_VALUE, max2=Integer.MIN_VALUE;
        for(int i=0; i<nums.length; i++){
            if(max2!=Integer.MIN_VALUE){
                if(nums[i]>max2) return true;
                else if(nums[i]>max1) max2=nums[i];
                else if(nums[i]<max1) max1=nums[i];
            }else{
                if(max1==Integer.MIN_VALUE) max1=nums[i];
                else if(max1>nums[i]) max1=nums[i];
                else if(max1<nums[i]) max2=nums[i];
            }
        }
        return false;
    }
}
```

当然，可以进一步地优化为：

```java
class Solution {
     public boolean increasingTriplet(int[] nums) {
        int first_num = Integer.MAX_VALUE;
        int second_num = Integer.MAX_VALUE;
        for (int n: nums) {
            if (n <= first_num) {
                first_num = n;
            } else if (n <= second_num) {
                second_num = n;
            } else {
                return true;
            }
        }
        return false;
    }
}
```

