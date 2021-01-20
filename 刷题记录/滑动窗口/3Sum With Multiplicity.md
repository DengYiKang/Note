# 3Sum With Multiplicity

### 问题

`arr`为整型数组，求`(i, j, k)`的个数，使得`arr[i]+arr[j]+arr[j]=target`，求三元组的个数

### 解决方案：滑动窗口，时间复杂度$O(n^2)$

很熟悉了，只不过这个题有一个注意点，就是滑动窗口的`lo，hi`，如果它们都指向了一个值相同的数，那么它们之间都是相同的数，因此计数规则不同。

```java
class Solution {
    public int threeSumMulti(int[] arr, int target) {
        int MOD=1_000_000_007;
        Arrays.sort(arr);
        int ans=0;
        for(int i=0; i<arr.length; i++){
            int need=target-arr[i];
            int lo=i+1, hi=arr.length-1;
            while(lo<hi&&lo>i&&hi<arr.length){
                int sum=arr[lo]+arr[hi];
                if(sum==need){
                    //注意这种情况
                    if(arr[lo]==arr[hi]){
                        ans+=(hi-lo+1)*(hi-lo)/2;
                        ans%=MOD;
                        break;
                    }
                    int lcnt=1, hcnt=1;
                    while(lo+1<hi&&arr[lo]==arr[lo+1]){
                        lcnt++;
                        lo++;
                    }
                    while(lo<hi-1&&arr[hi]==arr[hi-1]){
                        hcnt++;
                        hi--;
                    }
                    ans+=lcnt*hcnt;
                    ans%=MOD;
                    lo++;
                    hi--;
                }else if(sum>need){
                    hi--;
                }else{
                    lo++;
                }
            }
        }
        return ans;
    }
}
```

