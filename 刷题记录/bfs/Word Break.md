# Word Break

### 问题

输入一个字符串s与一个字典，问字典中的字符串组合成s？

### 解决方案：BFS，$O(N^2)$

假设$s[i...j]$在字典中能找到，那么$i$与$j$之间连一条线。在整个字符串s上按照上述规则可以构成一个图。对起点进行BFS，若能到达终点则有解。

> 另一种解法是dp，详情见dp目录

```java
class Solution {
    
    public boolean wordBreak(String s, List<String> wordDict) {
        Queue<Integer> queue=new LinkedList<>();
        HashSet<Integer> vis=new HashSet<>();
        HashSet<String> dict=new HashSet<>();
        for(String str:wordDict){
            dict.add(str);
        }
        queue.offer(0);
        while(!queue.isEmpty()){
            int pos=queue.poll();
            if(vis.contains(pos)) continue;
            vis.add(pos);
            for(int i=pos; i<s.length(); i++){
                String sub="";
                if(i==pos) sub=String.valueOf(s.charAt(pos));
                else sub=s.substring(pos, i+1);
                if(dict.contains(sub)){
                    if(i+1>=s.length()) return true;
                    queue.offer(i+1);
                }
            }
        }
        return false;
    }
}
```