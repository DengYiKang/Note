# Bitwise ORs of Subarrays

### 问题

`arr`为整型数组，对每个连续的子数组`sub=[arr[i], arr[i+1], ..., arr[j]], i<=j`，对它们按位或`arr[i]|...|arr[j]`，问总共有多少这种值。

### 解决方案：dp优化，时间复杂度$O(nlogm)$，$m$为数组元素的最大位数

设`result[i, j]`表示`arr[i]|...|arr[j]`，那么有`result[i, j]=result[i, j-1]|arr[j]`。

其中`i`取`0～j`。但注意到，变化的只有`j`。因此我们可以对给定的`j`对应的所有的`result[i,j], (i<=j)`视为一个集合`set[j]`，那么就有`set[j]=set[j-1]|arr[j]`，发现每个集合与`arr[j]`进行运算后能够转换为下一阶段的集合。

```java
class Solution {
    public int subarrayBitwiseORs(int[] arr) {
        HashSet<Integer> ans=new HashSet<>();
        HashSet<Integer> tmp=new HashSet<>();
        tmp.add(0);
        for(int x:arr){
            HashSet<Integer> next=new HashSet<>();
            for(int y:tmp){
                next.add(y|x);
            }
            //注意i==j的情况
            next.add(x);
            tmp=next;
            ans.addAll(tmp);
        }
        return ans.size();
    }
}
```

