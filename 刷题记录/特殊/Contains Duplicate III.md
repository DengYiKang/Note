# Contains Duplicate III

### 问题

输入一个整型数组，判断是否存在一组数$nums[i] ，nums[j]$，使得$|nums[i]-nums[j]|<=t，|i-j|<=k$。

### 解决方案

令w=t+1，在数轴上每w长划分一个桶，对于两个数nums[i]，nums[j]，若它们符合题意，那么：

+ 它们在同一个桶中
+ 它们在相邻的桶中

注意到，当同一个桶中有两个数时即可得到解，因此在遍历的过程中，每个桶最多有一个数。因此用一个map建立索引。

```java
class Solution {
    
    //注意正负数的映射不同
    long getId(long i, long w){
        return i<0?(i+1)/w-1:i/w;
    }
    
    public boolean containsNearbyAlmostDuplicate(int[] nums, int k, int t) {
        if(t<0) return false;
        HashMap<Long, Long> mp=new HashMap<>();
        long w=(long)t+1;
        for(int i=0; i<nums.length; i++){
            long id=getId(nums[i], w);
            if(mp.containsKey(id)) return true;
            if(mp.containsKey(id-1)&&Math.abs(mp.get(id-1)-nums[i])<w) return true;
            if(mp.containsKey(id+1)&&Math.abs(mp.get(id+1)-nums[i])<w) return true;
            mp.put(id, (long)nums[i]);
            if(i>=k) mp.remove(getId(nums[i-k], w));
        }
        return false;
    }
}
```

