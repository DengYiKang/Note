# Queue Reconstruction by Height

### 问题

现有一个人群队列，`(h,k)`表示一个人的状态，`h`表示这个人的高度，`k`表示这个人的前方站着比这个人高或等高的人有`k`个。先给一个数组`int [][]people`，乱序，要求复原出这个队列的状态。

**Example**

```
Input:
[[7,0], [4,4], [7,1], [5,0], [6,1], [5,2]]

Output:
[[5,0], [7,0], [5,2], [6,1], [4,4], [7,1]]
```

### 解决方案

从身高最小的开始考虑，注意到第一个考虑的人的状态是`(h,k)`，那么他在队列的位置是第`k`位（从0开始计数）。依次类推，若某个位置已经在不同阶段计算时分配了，那么顺移即可。

注意，同一身高的人必须在同一个阶段考虑，同一个阶段是指`k`会相互影响，也即这个阶段对于之前考虑的等高人占了的位置，不应顺移，应该把这个位置纳入计数中。

```java
class Solution {
    public int[][] reconstructQueue(int[][] people) {
        if(people==null||people.length==0||people[0].length==0) return new int[][]{};
        //排序用的
        List<int[]> list=new ArrayList<>();
        //为后面重组用的
        HashMap<Integer, int[]> mp=new HashMap<>();
        for(int i=0; i<people.length; i++){
            list.add(people[i]);
        }
        //排序，身高低的在前，等高下k小在前
        Collections.sort(list, new Comparator<int[]>(){
           @Override
            public int compare(int[] a, int[] b){
                if(a[0]!=b[0]) return a[0]-b[0];
                else return a[1]-b[1];
            }
        });
        //判断前面的人是否等高，即是否在同一阶段，cnt表示该阶段已经考虑的等高人数
        int pre=-1, cnt=0;
        for(int[] p:list){
            if(pre==p[0]) cnt++;
            else cnt=0;
            //等高的人所占的位置应该纳入计数中，因此减去cnt
            int preCnt=p[1]+1-cnt, i=0;
            while (i<people.length) {
                //不同阶段占掉的位置，顺移，不纳入计数
                while(mp.containsKey(i)) i++;
                preCnt--;
                if(preCnt==0) break;
                i++;
            }
            mp.put(i, p);
            pre=p[0];
        }
        int[][] ans=new int[people.length][2];
        for(int i=0; i<people.length; i++){
            ans[i][0]=mp.get(i)[0];
            ans[i][1]=mp.get(i)[1];
        }
        return ans;
    }
}
```

