# Exclusive Time of Functions

### 问题

模拟单线程cpu，给一个logs日志，记录各种任务的执行信息，输出各种任务执行的总时长。

**Example 1:**

<img src="../pic/12.png" alt="img" style="zoom:67%;" />

```
Input: n = 2, logs = ["0:start:0","1:start:2","1:end:5","0:end:6"]
Output: [3,4]
```

### 解决方案

#### 一、维护logStack和sumStack

因为有新的任务插入，因此在stack里的弹出的两个time的差值还要减去其中的其他任务执行的时长。因此对于每个任务开始时，为它分配一个变量sum来记录，结束时从sumStack中弹出。

```java
class Solution {
    public int[] exclusiveTime(int n, List<String> logs) {
        Stack<int[]> stack=new Stack<>();
        Stack<Integer> sumStack=new Stack<>();
        sumStack.push(0);
        int[] ans=new int[n];
        for(String log:logs){
            String[] splits=log.split(":");
            int id=Integer.valueOf(splits[0]);
            int type=splits[1].equals("start")?0:1;
            int timeStamp=Integer.valueOf(splits[2]);
            if(stack.isEmpty()||type==0) {
                stack.push(new int[]{id, type, timeStamp});
                sumStack.push(0);
            }
            else{
                int[] top=stack.pop();
                int during=timeStamp-top[2]+1-sumStack.pop();
                ans[id]+=during;
                int tmp=sumStack.pop();
                tmp+=timeStamp-top[2]+1;
                sumStack.push(tmp);
            }
        }
        return ans;
    }
}
```

#### 二、

栈顶的任务id就是当前执行的任务id，因此维护一个cur_time变量，当一个时间结点new_time到来时，将上述两时间点之间的长度加在栈顶任务id的时长中。要注意new_time的类型不同（即开始还是结束），操作也不同。

```java
class Solution {
    public int[] exclusiveTime(int n, List<String> logs) {
        Stack<Integer> stack=new Stack<>();
        int[] ans=new int[n];
        String[] first_splits=logs.get(0).split(":");
        stack.push(Integer.valueOf(first_splits[0]));
        int cur_time=Integer.valueOf(first_splits[2]);
        for(int i=1; i<logs.size(); i++){
            String[] splits=logs.get(i).split(":");
            int id=Integer.valueOf(splits[0]);
            int type=splits[1].equals("start")?0:1;
            int time=Integer.valueOf(splits[2]);
            if(!stack.isEmpty()) ans[stack.peek()]+=time-cur_time;
            cur_time=time;
            if(type==0) stack.push(id);
            else {
                ans[stack.peek()]++;
                cur_time++;
                stack.pop();
            }
        }
        return ans;
    }
}
```

