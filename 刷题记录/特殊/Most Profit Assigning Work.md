# Most Profit Assigning Work

### 问题

有一组任务，`difficulty[i]`和`profit[i]`分别表示第`i`个任务的难度和报酬；有一组工人，`worker[i]`表示第`i`个工人能处理的最大难度。每个工人至多分配一个任务，一个任务可以执行多次，问最大报酬是多少。

### 解决方案：

将`difficulty`和`profit`进行排序，难度小的在前，如果难度相等，报酬高的在前。然后对它们进行筛选，筛除那些可由同等难度或更低难度的报酬更高的任务替代的任务。

将`worker`升序排序，两个指针指向任务和worker，同时移动。

```java
class Solution {
    public int maxProfitAssignment(int[] difficulty, int[] profit, int[] worker) {
        List<int[]> list=new ArrayList<>();
        for(int i=0; i<profit.length; i++){
            list.add(new int[]{difficulty[i], profit[i]});
        }
        Collections.sort(list, (a, b)->{
            if(a[0]==b[0]) return b[1]-a[1];
            else return a[0]-b[0];
        });
        Arrays.sort(worker);
        int pre=list.get(0)[1];
        HashSet<Integer> dump=new HashSet<>();
        for(int i=1; i<list.size(); i++){
            if(pre>=list.get(i)[1]) dump.add(i);
            else pre=list.get(i)[1];
        }
        int task=list.size()-1;
        int pos=worker.length-1;
        int res=0;
        while(pos>=0&&task>=0){
            while(task>=0&&
                  (dump.contains(task)||list.get(task)[0]>worker[pos]))
                task--;
            if(task<0) break;
            res+=list.get(task)[1];
            pos--;
        }
        return res;
    }
}
```

