# Number of Boomerangs

### 问题

在二维坐标系下，现有`n`个不同的点，`points[i]=[x_i, y_i]`，定义三元组`(i, j, k)`，其中`i`到`j`的距离与`j`到`k`的距离相等。求这类三元组的个数。

### 解决方案：时间复杂度$O(n^2)$，空间复杂度$O(n)$

对于一个点`i`，把所有与点`i`等距的点`j`全部划分到一个集合`s`中，那么在这个集合`s`中满足题意的三元组的个数为$2*C_{|s|}^2=|s|*(|s|-1)$。

```java
class Solution {
    public int numberOfBoomerangs(int[][] points) {
        HashMap<Integer, Integer> mp=new HashMap<>();
        int ans=0;
        for(int i=0; i<points.length; i++){
            for(int j=0; j<points.length; j++){
                if(i==j) continue;
                int d=getDistance(i, j, points);
                mp.put(d, mp.getOrDefault(d, 0)+1);
            }
            for(int value:mp.values()){
                ans+=value*(value-1);
            }
            mp.clear();
        }
        return ans;
    }
    int getDistance(int a, int b, int[][] points){
        int x=points[a][0]-points[b][0];
        int y=points[a][1]-points[b][1];
        return x*x+y*y;
    }
}
```

