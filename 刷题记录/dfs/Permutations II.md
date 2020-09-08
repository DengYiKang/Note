# Permutations II

### 问题

给一个拥有重复元素的数组，求其所有的排列，不能重复。

### 解决方案

主要是巩固dfs中避免重复的技巧。跟./Combination Sum II类似，只不过前者从pos+1位置开始搜索，这里从头搜索，需要加个first表示在这趟dfs中是否是搜索的第一个。

```java
class Solution {
    List<List<Integer>> ans;
    List<Integer> list;
    boolean[] vis;
    public List<List<Integer>> permuteUnique(int[] nums) {
        ans=new ArrayList<>();
        list=new ArrayList<>();
        vis=new boolean[nums.length];
        int pre=0;
        //注意别忘了排序
        Arrays.sort(nums);
        for(int i=0; i<nums.length; i++){
            if(i!=0&&nums[pre]==nums[i]) continue;
            dfs(i, 0, nums);
            pre=i;
        }
        return ans;
    }
    void dfs(int pos, int len, int[] nums){
        list.add(nums[pos]);
        vis[pos]=true;
        len++;
        if(len==nums.length){
            ans.add(new ArrayList<>(list));
            vis[pos]=false;
            list.remove(list.size()-1);
            return;
        }
        int pre=pos;
        //在这个循环中，如果搜索的第二个跟第一个重复，则剪枝
        boolean first=true;
        for(int i=0; i<nums.length; i++){
            if(vis[i]) continue;
            if(!first&&nums[pre]==nums[i]){
                first=false;
                continue;
            }
            dfs(i, len, nums);
            pre=i;
            //注意别忘了
            first=false;
        }
        vis[pos]=false;
        list.remove(list.size()-1);
    }
}
```

