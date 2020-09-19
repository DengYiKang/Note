#  Minimum Size Subarray Sum

### 问题

输入一个元素为正数的数组A，与一个正数s。找到一个最小的长度len，使得长度为len的A的连续子序列的和大于s。

### 解决方案：

设序列为ab...c....d...ef，从左到右扫描，假设a->f的长度恰好大于等于s，那么c->d的长度一定小于s，但c->f的长度可能符合要求。

#### 滑动窗口，$O(n)$

```java
class Solution {
    public int minSubArrayLen(int s, int[] nums) {
        if(nums.length==0) return 0;
        int lo=0, hi=0, sum=nums[0], ans=Integer.MAX_VALUE;
        while(lo<=hi&&hi<nums.length){
            if(sum<s) {
                hi++;
                if(hi<nums.length) sum+=nums[hi];
                else break;
            }
            else if(sum>=s) {
                sum-=nums[lo];
                ans=Math.min(ans, hi-lo+1);
                lo++;
            }
        }
        return ans==Integer.MAX_VALUE?0:ans;
    }
}
```

#### dp，$O(n)$ 

设$dp[i]$表示以$i$为起点的满足和的要求的最小长度。$cnt[i]$表示$dp[i]$对应的子序列的和，$mp[i]$表示$dp[i]$对应的子序列的末端(包括$mp[i]$)。

已知$dp[i]$，计算$dp[i+1]$时，要考虑三种情况：

+ $mp[i]<=i+1$:此时要从$i+1$开始搜索
+ $mp[i]>i+1$：此时从$A[i...mp[i]]$开始搜索

```java
class Solution {
    public int minSubArrayLen(int s, int[] nums) {
        int len=nums.length, ans=Integer.MAX_VALUE;
        if(len==0) return 0;
        int[] dp=new int[len];
        int[] cnt=new int[len];
        HashMap<Integer, Integer> mp=new HashMap<>();
        Arrays.fill(dp, Integer.MAX_VALUE);
        for(int i=0, sum=0; i<len; i++){
            sum+=nums[i];
            if(sum>=s){
                dp[0]=i+1;
                cnt[0]=sum;
                mp.put(0, i);
                break;
            }
        }
        for(int i=1; i<len; i++){
            if(dp[i-1]==Integer.MAX_VALUE) break;
            int lo=mp.get(i-1);
            int pre=0, offset=0, sum=0;
            if(lo<=i) {
                pre=0;
                offset=i;
                sum=0;
            }
            else {
                pre=dp[i-1]-2;
                offset=lo;
                sum=cnt[i-1]-nums[i-1]-nums[mp.get(i-1)];
            }
            for(int j=0; j+offset<len; j++){
                sum+=nums[j+offset];
                if(sum>=s){
                    dp[i]=pre+j+1;
                    cnt[i]=sum;
                    mp.put(i, j+offset);
                    break;
                }
            }
        }
        for(int i=0; i<len; i++){
            ans=Math.min(dp[i], ans);
        }
        return ans==Integer.MAX_VALUE?0:ans;
    }
}
```

