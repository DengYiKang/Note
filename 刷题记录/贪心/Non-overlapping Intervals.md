# Non-overlapping Intervals

### 问题

现有一个区间的集合，移除其中的`k`个区间，使得该集合中的所有区间不重叠，求`k`的最小值。

### 解决方案：时间复杂度$O(nlogn)$，空间复杂度$O(n)$

对区间进行排序，使得区间左端小的在前，其次右端小的在前。维护一个集合`s`。

扫描区间，根据当前区间`a`与集合`s`里的最后一个区间`b`的关系判断，是把当前区间`a`替换末位区间`b`、还是添加区间`a`，还是不做操作。

```java
class Solution {
    public int eraseOverlapIntervals(int[][] intervals) {
        if(intervals==null||intervals.length==0) return 0;
        List<int[]> list=new ArrayList<>();
        for(int i=0; i<intervals.length; i++){
            list.add(new int[]{intervals[i][0], intervals[i][1]});
        }
        Collections.sort(list, new Comparator<int[]>(){
           @Override
            public int compare(int[] a, int[] b){
                if(a[0]!=b[0]) return a[0]-b[0];
                else return a[1]-b[1];
            }
        });
        List<int[]> ans=new ArrayList<>();
        for(int[] interval:list){
            int tail=ans.size();
            if(tail==0){
                ans.add(interval);
            }else{
                int[] pre=ans.get(tail-1);
                if(interval[1]<=pre[1]){
                    ans.remove(tail-1);
                    ans.add(interval);
                }else if(interval[0]>=pre[1]){
                    ans.add(interval);
                }
            }
        }
        return intervals.length-ans.size();
    }
}
```

