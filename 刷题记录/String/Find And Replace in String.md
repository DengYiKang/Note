# Find And Replace in String

### 问题

原字符串`S`，现在要求把`S`中的某部分替换。第`i`次操作中，如果在原串`S`中`indexes[i]`位置是以`sources[i]`开头的，那么把`sources[i]`替换成`targets[i]`。（注意是原串中！）

### 解决方案：时间复杂度$O(nlogn)$

注意，能否替换的判断是在原串`S`上进行的，因此`indexes`的相对位置没有关系（如果题目给的数据不冲突的话）。将`indexes`进行排序，方便后续拼接字符串。

```java
class Solution {
    public String findReplaceString(String S, int[] indexes, String[] sources, String[] targets) {
        List<int[]> list=new ArrayList<>();
        for(int i=0; i<indexes.length; i++){
            list.add(new int[]{i, indexes[i]});
        }
        Collections.sort(list, (a, b)->a[1]-b[1]);
        //能否替换的标志
        boolean[] success=new boolean[indexes.length];
        for(int i=0; i<indexes.length; i++){
            int len=sources[i].length();
            if(S.substring(indexes[i], indexes[i]+len).equals(sources[i])) success[i]=true;
        }
        int pre=0;
        StringBuilder sb=new StringBuilder();
        for(int i=0; i<list.size(); i++){
            int index=list.get(i)[0];
            int index_val=list.get(i)[1];
            if(success[index]){
                sb.append(S.substring(pre, index_val));
                sb.append(targets[index]);
                pre=index_val+sources[index].length();
            }
        }
        sb.append(S.substring(pre, S.length()));
        return sb.toString();
    }
}
```



