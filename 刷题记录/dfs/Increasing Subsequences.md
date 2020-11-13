# Increasing Subsequences

### 问题

现有一个整型数组，求所有升序子串。

### 解决方案：dfs，时间复杂度$O(n^2)$，空间复杂度$O(n^2)$

也是去重的dfs的例子。记得在主函数调用处做去重判断。

```java
class Solution {
    List<List<Integer>> ans;
    public List<List<Integer>> findSubsequences(int[] nums) {
        if(nums==null||nums.length==0) return new ArrayList<>();
        ans=new ArrayList<>();
        List<Integer> tmp=new ArrayList<>();
        //别忘了首位的去重
        HashSet<Integer> vis=new HashSet<>();
        for(int i=0; i<nums.length; i++){
            if(vis.contains(nums[i])) continue;
            vis.add(nums[i]);
            dfs(i, nums, -101, tmp);
        }
        return ans;
    }
    void dfs(int pos, int[] data, int lo, List<Integer> tmp){
        tmp.add(data[pos]);
        lo=data[pos];
        if(tmp.size()>1) ans.add(new ArrayList<>(tmp));
        //去重
        HashSet<Integer> vis=new HashSet<>();
        for(int i=pos+1; i<data.length; i++){
            if(data[i]<lo||vis.contains(data[i])) continue;
            vis.add(data[i]);
            dfs(i, data, data[i], tmp);
        }
        vis.clear();
        tmp.remove(tmp.size()-1);
    }
}
```

