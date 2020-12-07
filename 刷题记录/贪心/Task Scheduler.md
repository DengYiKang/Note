# Task Scheduler

### 问题

模拟cpu的任务执行序列，`tasks[1...n]`的元素为`A~Z`，表示任务。现规定相同任务之间必须要有`n`个slots，若无任务执行，则插入`idle`，问对于`tasks`数组，包含`idle`在内的序列长度至少为多少？

### 解决方案：贪心，时间复杂度$O(n^2)$

从频数最大的任务考虑，假设频数最大的为`A`，其频数为`f`，那么每个`A`之间至少有`n`个slots。则有：

`A##...##A##...##A`

那么有`slots=n*(f-1)`。

若有多个最大的频数的任务，共有`m`个这样的任务，那么可以依次排列在`A`的后面，如：

`ABC##...##ABC##...##ABC`

那么有`slots=(n-m+1)*(f-1)`。

注意到剩余的任务数的频数必然<=`f-1`，即可以把它们分别放在不同的`A`的间隔中。

设其他任务总数为`other_sum`

那么有：`idles=slots-(other_sum)`

```java
class Solution {
    public int leastInterval(char[] tasks, int n) {
        int[] cnt=new int[26];
        for(char ch:tasks){
            cnt[ch-'A']++;
        }
        int max_cnt=-1, max_val_nums=0;
        for(int i=0; i<26; i++){
            if(cnt[i]>max_cnt){
                max_cnt=cnt[i];
                max_val_nums=1;
            }else if(cnt[i]==max_cnt){
                max_val_nums++;
            }
        }
        int slots=(n-max_val_nums+1)*(max_cnt-1);
        int otherSum=tasks.length-max_val_nums*max_cnt;
        int idles=Math.max(0, slots-otherSum);
        return idles+tasks.length;
    }
}
```

