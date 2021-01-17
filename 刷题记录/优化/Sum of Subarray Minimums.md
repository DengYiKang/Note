# Sum of Subarray Minimums

### 问题

`A`为整型数组，求`sum(min(B))`，其中`B`是`A`的连续子数组。

### 解决方案：时间复杂度$O(n)$

跟Bitwise ORs of Subarrays的题目的思路很类似。

假设`result[i,j]`为`min(A[i], A[i+1], ..., A[j])`，那么有`result[i, j]=min(result[i, j-1], A[j]) `，那么将`result[i, j]`对应的所有集合设为`set[j]`，那么有`set[j]=min(set[j-1], A[j])`。对`j`进行的每次遍历，可以同步地把`set[j-1]`转换为`set[j]`。因为有很多重复元素，可以记录元素以及对应的出现次数。

同时要注意到`set[j]`是递增的，因此可以用`stack`来更新需要更新的（因为递增，所以只需要更新栈顶的比`A[j]`大的元素），这将时间复杂度降至$O(n)$。

```java
class Solution {
    public int sumSubarrayMins(int[] arr) {
        int MOD=1_000_000_007;
        int ans=0, sum=0;
        Stack<Node> stack=new Stack<>();
        for(int i=0; i<arr.length; i++){
            int cnt=1;
            while(!stack.isEmpty()&&stack.peek().val>=arr[i]){
                Node node=stack.pop();
                cnt+=node.cnt;
                sum-=node.cnt*node.val;
            }
            stack.push(new Node(arr[i], cnt));
            sum+=arr[i]*cnt;
            ans+=sum;
            ans%=MOD;
        }
        return ans;
    }
    class Node{
        public int val;
        public int cnt;
        Node(int v, int c){
            val=v;
            cnt=c;
        }
    }
}
```

