# Number of Longest Increasing Subsequence

### 问题

求最长升序子序列的数量。

### 解决方案：dp，时间复杂度$O(n^2)$

`len[i],count[i]`分别表示以第`i`位结尾的LIS的长度和对应数量。

则有：
$$
len[i]=max\{len[j]+1, len[i]\}, if\;j<i\;and\;A[j]<A[i]\\
count[i]+=count[j],\;if\;j<i\;and\;len[i]=len[j]+1\;and\;A[j]<A[i]
$$
`count[i]`的那个条件表示当前位结尾的LIS的长度为整体的LIS的长度。

```java
class Solution {
    public int findNumberOfLIS(int[] nums) {
        int[] len=new int[nums.length];
        Arrays.fill(len, 1);
        int[] count=new int[nums.length];
        int max_len=0;
        for(int i=0; i<nums.length; i++){
            for(int j=0; j<i; j++){
                if(nums[i]>nums[j]){
                    len[i]=Math.max(len[i], len[j]+1);
                }
            }
            for(int j=0; j<i; j++){
                if(len[j]+1==len[i]&&nums[i]>nums[j]){
                    count[i]+=count[j];
                }
            }
            if(count[i]==0) count[i]=1;
            max_len=Math.max(max_len, len[i]);
        }
        int ans=0;
        for(int i=0; i<nums.length; i++){
            if(len[i]==max_len) ans+=count[i];
        }
        return ans;
    }
    
}
```



