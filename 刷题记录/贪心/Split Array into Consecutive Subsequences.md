# Split Array into Consecutive Subsequences

### 问题

现有一个升序的整型数组`nums`，问是否存在一种划分，使得所有子数组的元素都是连续的，要求子数组的最小长度为3。

### 解决方案：贪心，时间复杂度$O(n)$ 

对于第`i`个数`nums[i]`，有两种操作：

+ `make_set`：以`nums[i]`为起点构造一个子数组
+ `join`：加入到末尾元素为`nums[i]-1`的子数组中，成为末尾元素为`nums[i]`的子数组

将各个已产生的末尾元素看成资源。如果`make_set`优先于`join`，因为`nums`是升序的，那么可能会导致一些资源浪费（因为要求子数组是连续的，那么一些末尾元素会因为升序而失效）。因此`join`的操作优先于`make_set`。

```java
class Solution {
    public boolean isPossible(int[] nums) {
        if(nums==null||nums.length==0) return false;
        boolean[] vis=new boolean[nums.length];
        HashMap<Integer, Integer> mp=new HashMap<>();
        for(int i=0; i<nums.length; i++){
            if(vis[i]) continue;
            //优先执行join操作
            int num=mp.getOrDefault(nums[i]-1, 0);
            if(num!=0){
                mp.put(nums[i]-1, num-1);
                mp.put(nums[i], mp.getOrDefault(nums[i], 0)+1);
            }else if(!make_set(i, nums, vis, mp)){
                //join操作失败，再执行make_set操作
                return false;
            }
        }
        return true;
    }
    public boolean make_set(int index, int[] nums, boolean[] vis, HashMap<Integer, Integer> mp){
        int second=-1;
        int third=-1;
        int i=index+1;
        while((second==-1||third==-1)
              &&i<nums.length&&nums[i]<=nums[index]+2){
            if(!vis[i]&&second==-1&&nums[i]==nums[index]+1) {
                second=i;
            }
            if(!vis[i]&&third==-1&&nums[i]==nums[index]+2) {
                third=i;
            }
            i++;
        }
        if(second!=-1&&third!=-1){
            vis[second]=vis[third]=true;
            mp.put(nums[third], mp.getOrDefault(nums[third], 0)+1);
            return true;
        }
        else return false;
    }
}
```

