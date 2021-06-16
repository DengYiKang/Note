# Capacity To Ship Packages Within D Days

### 问题

需要按顺序运输一堆货物，第i个货物的重为weights[i]，一天最多运capacity重量。问用D天运输（这D天必须用到，不多不少），现在给定weights与D，求capacity的最小值。注意这些必须按照规定的顺序运（例如运weight[10]之前必须运weight[1]）。

### 解决方案：二分搜索，时间复杂度$O(nlogm)$

时间复杂度里，$n$为weights的长度，$w$为weights的总和。

```java
class Solution {
        public int shipWithinDays(int[] weights, int D) {
                int sum=0;
                for(int i=0; i<weights.length; i++){
                        sum+=weights[i];
                }
                int l=0, r=sum;
                while(l<=r&&r<=sum){
                        int mid=(l+r)/2;
                        if(canHold(weights, D, mid)) r=mid-1;
                        else l=mid+1;
                }
                return l;
        }
        boolean canHold(int[] weights, int D, int capacity){
                int d=1, tmp_sum=0;
                for(int i=0; i<weights.length; i++){
                        if(weights[i]>capacity) return false;
                        if(tmp_sum+weights[i]>capacity){
                                d++;
                                tmp_sum=0;
                        }
                        tmp_sum+=weights[i];
                }
                return d<=D;
        }
}
```

