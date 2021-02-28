# Delete Columns to Make Sorted II

### 问题

输入`String[] strs`，所有的`strs[i]`具有相同的长度。一个索引集合`{i, j, k, ...}`，对`strs`数组中的所有字符串将索引集合中的所有索引对应的字符删去，使得`strs`成字典序（升序），求索引集合的最小size。

### 解决方案

如果各个字符串的首位成严格的字典序，那么strs成字典序。

因此从高到低进行比较，假设有`strs[0][0]<strs[1][0]<...<strs[i][0]=strs[i+1][0]<....`，其中第i位与第i+1位待定，那么往后我们只需要比较第i个位置和第i+1个位置，因为在后续的删除操作中，除了i和i+1，无论怎么删除，其余的字符串的首位已经成严格字典序了，那么它们必定成字典序。

因此维护一个队列，初始值为`0~len(strs)-2`，（因为是比较相邻的，因此只需存低位）。

在比较的过程中，如果当前位的比较相等，那么加入队列。

如果当前位成逆序，那么将队列回复到未比较前的状态。

```java
class Solution {
    public int minDeletionSize(String[] strs) {
            int len=strs[0].length();
            int depth=0, del=0;
            Queue<Integer> queue=new LinkedList<>();
            for(int i=0; i<strs.length-1; i++){
                    queue.offer(i);
            }
            while(!queue.isEmpty()&&depth<len){
                    int size=queue.size();
                    Queue<Integer> back=new LinkedList<>(queue);
                    for(int i=0; i<size; i++){
                            int index=queue.poll();
                            char a=strs[index].charAt(depth);
                            char b=strs[index+1].charAt(depth);
                            if(a>b){
                                    queue.clear();
                                    del++;
                                    queue=null;
                                    queue=back;
                                    break;
                            }else if(a==b){
                                    queue.offer(index);
                            }
                    }
                    depth++;
            }
            return del;
    }
}
```

