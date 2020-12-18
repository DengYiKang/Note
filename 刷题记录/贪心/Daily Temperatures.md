# Daily Temperatures

### 问题

`T`是整型数组，求一个数组`ans`，其中`ans[i]`表示`arg_min(ans[i]){T[i+ans[i]]>T[i]}`，且`ans[i]>=0`。即在与右方距离自己最近的比自己大的数之间距离。如果没有，怎`ans[i]=0`。

### 解决方案：贪心+stack，时间复杂度$O(n)$ 

从右往左遍历，维护一个待比较的索引池，设当前索引为`i`，如果`T[i]`比池中所有的索引对应的数都大，那么将池清空，`ans[i]=0`，将`i`加入到索引池。注意到，在索引池中，最近加入的索引对应的数必然是池中最小的，并且其索引也最小。因此，对当前索引`j`，将池中比`T[j]`小的数全部剔除，池中第一个大的数的索引恰好是池中最小的，直接相减即可。

这个索引池可以用stack来模拟。

```java
class Solution {
    public int[] dailyTemperatures(int[] T) {
        int[] ans = new int[T.length];
        Stack<Integer> stack = new Stack();
        for (int i = T.length - 1; i >= 0; --i) {
            while (!stack.isEmpty() && T[i] >= T[stack.peek()]) stack.pop();
            ans[i] = stack.isEmpty() ? 0 : stack.peek() - i;
            stack.push(i);
        }
        return ans;
    }
}
```

