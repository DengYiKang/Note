# 132 Pattern

### 问题

现有一个数组`nums`，问能否找到索引`i<j<k`使得`nums[i]<nums[k]<nums[j]`。

### 解决方案：

#### 找特征：时间复杂度$O(n^2)$，空间复杂度$O(n)$

把上升的区间加入到list中，把待观察的点与list中的所有区间比对，若该点在list中的某个区间内，则返回`true`

#### 利用min数组以及从右往左扫描，保持顺序性：时间复杂度$O(n)$，空间复杂度$O(n)$

维护一个`min[1...n]`数组，`min[i]`表示区间`[0,i]`上的最小数，那么`min[i]`所表示的数必然在区间`[0,i]`上。为了保持顺序性，考虑从右往左扫描，对于待考虑的数`nums[j]`，只需保证`nums[j]>min[j]`即可，这样大小关系和顺序关系都得到了保证。对于`k`，只需用一个`stack`来保存。注意，当`nums[j]>nums[k]`时，直接返回`true`，否则圧栈，因此栈中的数都是有序的，即栈顶的数为栈中最小数，因此比较`nums[j]`与`nums[k]`间的关系时，只需要比较`nums[j]`与栈顶的数即可。

```java
class Solution {
    public boolean find132pattern(int[] nums) {
        if(nums==null||nums.length<3) return false;
        int[] min=new int[nums.length];
        min[0]=nums[0];
        for(int i=1; i<nums.length; i++){
            min[i]=Math.min(min[i-1], nums[i]);
        }
        Stack<Integer> stack=new Stack<>();
        for(int i=nums.length-1; i>=0; i--){
            if(nums[i]>min[i]){
                while(!stack.isEmpty()&&stack.peek()<=min[i]) stack.pop();
                if(!stack.isEmpty()&&stack.peek()<nums[i]) return true;
                stack.push(nums[i]);
            }
        }
        return false;
    }
}
```





