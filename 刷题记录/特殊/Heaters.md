# Heaters

### 问题

在一维坐标上，有多个房子和加热器，每个加热器能加热的半径都是`r`，求把所有房子覆盖的最小`r`

### 解决方案：时间复杂度$O(n)$，空间复杂度$O(n)$

分段考虑，每次考虑两个相邻的加热器与其间的房子。

```java
class Solution {
    public int findRadius(int[] houses, int[] heaters) {
        if(houses==null||houses.length==0) return 0;
        List<int[]> list=new ArrayList<>();
        for(int i=0; i<houses.length; i++){
            list.add(new int[]{houses[i], 0});
        }
        for(int i=0; i<heaters.length; i++){
            list.add(new int[]{heaters[i], 1});
        }
        Collections.sort(list, (a, b)->a[0]-b[0]);
        int pre=-1, r=0;
        List<Integer> midList=new ArrayList<>();
        for(int[] cur:list){
            if(cur[1]==0) midList.add(cur[0]);
            else{
                for(int pos:midList){
                    int t_r=pre!=-1?Math.min(pos-pre, cur[0]-pos):cur[0]-pos;
                    r=Math.max(r, t_r);
                }
                midList.clear();
                pre=cur[0];
            }
        }
        if(midList.size()!=0){
            r=Math.max(r, midList.get(midList.size()-1)-pre);
        }
        return r;
    }
}
```

