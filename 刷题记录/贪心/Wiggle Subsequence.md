# Wiggle Subsequence

### 问题

寻找波动子序列(相邻数的差组成的数列正负相间)的最大长度。

### 解决方案：贪心，时间复杂度$O(n)$，空间复杂度$O(1)$

这题当然也可以用dp来做，`dp[i][j]`表示以`j`为尾的波动子序列当前状态为`i`的最大长度，时间复杂度为$O(n^2)$。

```java
class Solution {
    public int wiggleMaxLength(int[] nums) {
        if(nums==null||nums.length==0) return 0;
        if(nums.length==1) return 1;
        int start=0;
        //首两个必须不同
        while(start+1<nums.length&&nums[start]==nums[start+1]) start++;
        if(start+1==nums.length) return 1;
        //状态
        boolean dir=nums[start]-nums[start+1]>0;
        int cnt=2, tail=start+1;
        for(int i=start+2; i<nums.length; i++){
            if(dir){
                if(nums[i]>nums[tail]){
                    cnt++;
                    tail=i;
                    dir=false;
                }else if(nums[i]<nums[tail]){
                    //发展潜力更大，替换tail
                    tail=i;
                }
            }else{
                if(nums[i]<nums[tail]){
                    cnt++;
                    tail=i;
                    dir=true;
                }else if(nums[i]>nums[tail]){
                    //发展潜力更大，替换tail
                    tail=i;
                }
            }
        }
        return cnt;
    }
}
```

