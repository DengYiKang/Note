# Max Chunks To Make Sorted

### 问题

`arr`是一个整型数组，`arr[0...n]`对应`0~n`的某种排列。现对其进行划分，使得对每个子数组排序后，整个数组是个有序状态，求问最大的划分数是多少？

### 解决方案：滑动窗口，时间复杂度$O(n)$

假设窗口维护这一个划分，两端点为`left, right`。那么`arr[left...right]`为`left~right`的一个排列。

即`right=max(arr[left], ..., arr[right])`。排列的右端点保证了，那么剩下的呢？注意到，`left`是`arr[left,...n]`中的最小数，那么窗口里面都是所有比`left`大的数，且所有比`right`小的数都在窗口里面，因此该窗口与`arr[left,...right]`一一对应，即该窗口是一个合法的划分。

```java
class Solution {
    public int maxChunksToSorted(int[] arr) {
        int left=0, right=0, ans=0;
        while(left<=right&&right<arr.length){
            int max_pos=arr[right];
            while(right<arr.length&&right<max_pos){
                right++;
                max_pos=Math.max(max_pos, arr[right]);
            }
            right++;
            left=right;
            ans++;
        }
        return ans;
    }
}
```

