# Combination Sum II

### 问题

给一数组A[1...len]，一值target，要求输出数组A中所有和为targe组合，每个元素只能用一次，且不能重复。

### 解决方案

很容易想到dfs。这里的有趣之处在于不能重复。

```java
class Solution {
    private List<List<Integer>> ans;
    private List<Integer> list;
    
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        Arrays.sort(candidates);
        ans=new ArrayList<>();
        list=new ArrayList<>();
        int pre=0;
        for(int i=0; i<candidates.length; i++){
            //若candidates[i]==candidates[pre]，则dfs(pre,...)包含dfs（i,...）的解
            if(i!=0&&candidates[i]==candidates[pre]) continue;
            dfs(i, 0, candidates, target);
            pre=i;
        }
        return ans;
    }
    
    void dfs(int pos, int sum, int[] a, int target){
        sum+=a[pos];
        list.add(a[pos]);
        if(sum==target){
            ans.add(new ArrayList<Integer>(list));
            list.remove(list.size()-1);
            return;
        }else if(sum>target){
            list.remove(list.size()-1);
            return;
        }
        int pre=pos+1;
        for(int i=pos+1; i<a.length; i++){
            //在前序列固定的情况下，若a[pre]==a[i]，则dfs（pre,...）包含dfs(i,...)的解
            if(i!=pos+1&&a[pre]==a[i]) continue;
            dfs(i, sum, a, target);
            pre=i;
        }
        list.remove(list.size()-1);
    }
}
```

