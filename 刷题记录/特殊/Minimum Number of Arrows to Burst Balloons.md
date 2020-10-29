# Minimum Number of Arrows to Burst Balloons

### 问题

现有多个区间$[x_{start},x_{end}]$，若一个点在多个区间的范围内，则称该点能代表这些区间。求最小点集的长度，使得能代表所有区间。

### 解决方案：时间复杂度$O(n)$，空间复杂度$O(n)$

与合并多个重叠区间的题类似的思路。

对区间进行排序，维护一个区间列表，每次将待观察的区间与列表中的末位区间进行比较，取出它们的重叠部分来代替末位区间，若无重叠部分，则直接加入。

```java
class Solution {
    public int findMinArrowShots(int[][] points) {
        List<int[]> data=new ArrayList<>();
        List<int[]> list=new ArrayList<>();
        for(int i=0; i<points.length; i++){
            data.add(new int[]{points[i][0], points[i][1]});
        }
        Collections.sort(data, new Comparator<int[]>(){
            @Override
            public int compare(int[] a, int[] b){
                if(a[0]!=b[0]) return a[0]>b[0]?1:-1;
                else return a[1]>b[1]?1:-1;
            }
        });
        for(int[] interval:data){
            if(list.size()==0){
                list.add(interval);
            }else{
                int[] pre=list.get(list.size()-1);
                if(interval[1]<=pre[1]){
                    list.remove(list.size()-1);
                    list.add(interval);
                }else if(interval[0]<=pre[1]&&pre[1]<interval[1]){
                    list.remove(list.size()-1);
                    list.add(new int[]{interval[0], pre[1]});
                }else{
                    list.add(interval);
                }
            }
        }
        return list.size();
    }
}
```

