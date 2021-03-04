# Interval List Intersections

### 问题

有两个区间list，分别存储着多个互不相交的区间，已经排好序。求两个区间list的所有区间的重叠部分。

### 解决方案：时间复杂度$O(n)$

如果两个区间不重叠，那么将它们的四个端点排序，中间两个端点就是重叠区间的端点。

因为list中的区间都是互不相交且有序。因此维护两个索引`first_index`与`second_index`来指向正在比较的区间。

比较完后，谁的区间的末端更小，谁的索引更新+1。

```java
class Solution {
        public int[][] intervalIntersection(int[][] firstList, int[][] secondList) {
                int first_index=0;
                int second_index=0;
                List<int[]> list=new ArrayList<>();
                while(first_index<firstList.length&&second_index<secondList.length){
                        int a=firstList[first_index][0];
                        int b=firstList[first_index][1];
                        int c=secondList[second_index][0];
                        int d=secondList[second_index][1];
                        if(!(d<a||b<c)){
                                int[] sorted=new int[]{a, b, c, d};
                                Arrays.sort(sorted);
                                list.add(new int[]{sorted[1], sorted[2]});
                        }
                        if(b<d) first_index++;
                        else second_index++;
                        continue;
                }
                int[][] ans=new int[list.size()][2];
                for(int i=0; i<list.size(); i++){
                        ans[i]=list.get(i);
                }
                return ans;
        }
}
```

