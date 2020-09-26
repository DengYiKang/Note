# Find the Duplicate Number

### 问题

输入一个长度为`n+1`的数组`nums`，元素在`[1,n]`范围内，有且只有一种元素是重复的，求该元素

### 解决方案：循环检测，时间复杂度$O(n)$，空间复杂度$O(1)$

注意`nums`数组可以视为链表：

![pic](../pic/5.png)

因此可以用循环检测算法：

```java
class Solution {
    public int findDuplicate(int[] nums) {
        int first=nums[0], second=nums[0];
        do{
            first=nums[first];
            second=nums[nums[second]];
        }while(first!=second);
        first=nums[0];
        while(first!=second){
            first=nums[first];
            second=nums[second];
        }
        return first;
    }
}
```

