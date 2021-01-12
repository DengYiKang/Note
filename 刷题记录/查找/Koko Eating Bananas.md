# Koko Eating Bananas

### 问题

有几堆香蕉，一个猴子需要在`h`个小时内吃完这些香蕉，猴子每小时能吃`k`个香蕉，如果在一个小时内，猴子吃完当前堆，那么猴子在这一小时的剩余时间内不会去吃另一个堆，会等待新一个小时去吃新的堆。求问`k`的最小值是多少。

### 解决方案：二分查找，时间复杂度$O(nlog(w))$，$w$是堆的最大香蕉数

设`possible(m)`表示当`k`取`m`时，猴子能否在`h`小时内吃完。

注意到`possible(i)`是连续有序的，即`possible(i)`的排列为`false, false, false, ..., false, true, true, ..., true`。

因此可以对`possible(i)`进行二分查找，找到最小的`i`使得`possible(i)`为true。

```java
class Solution {
    public int minEatingSpeed(int[] piles, int H) {
        int max_val=0;
        for(int pile:piles){
            max_val=Math.max(max_val, pile);
        }
        int l=1, r=max_val;
        while(r>=l){
            int mid=(l+r)/2;
            if(possible(piles, H, mid)) r=mid-1;
            else l=mid+1;
        }
        return l;
    }
    public boolean possible(int[] piles, int h, int k){
        int need=0;
        for(int i=0; i<piles.length; i++){
            need+=(piles[i]-1)/k+1;
        }
        return need<=h;
    }
    
}
```

