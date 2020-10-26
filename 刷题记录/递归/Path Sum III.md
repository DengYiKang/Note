# Path Sum III

### 问题

现有一颗二叉树，一个和`target`，要求找到在树上找到一条路径，使得权值和为`target`，路径的起点与终点没有限制，求路径的总数。

### 解决方案：

#### 递归，时间复杂度$worstCase\;O(n^2)\;bestCase\;O(nlogn)$

```java
class Solution {
    public int pathSum(TreeNode root, int sum) {
        if(root==null) return 0;
        return findPath(root, sum)+pathSum(root.left, sum)+pathSum(root.right, sum);
    }
    public int findPath(TreeNode root, int sum){
        if(root==null) return 0;
        int res=0;
        if(sum==root.val) res++;
        res+=findPath(root.left, sum-root.val)+findPath(root.right, sum-root.val);
        return res;
    }
}
```

#### 递归，前缀和，时间复杂度$O(n)$，空间复杂度$O(n)$

假设有三个结点`a,b,c`，当前搜索的结点为`c`，其位置为：`a------b-------c`

那么若`bc=target`，则有`sum(a,b)=sum(a,c)-target`。

因此考虑维护一个`map`，使得key值为`d`，value值为使得`sum(a,b)=d`的`b`点的个数。

```java
HashMap<Integer, Integer> preSum;
public int pathSum(TreeNode root, int sum) {
        preSum = new HashMap();
        preSum.put(0,1);
        return helper(root, 0, sum, preSum);
    }
    
    public int helper(TreeNode root, int currSum, int target) {
        if (root == null) {
            return 0;
        }
        
        currSum += root.val;
        int res = preSum.getOrDefault(currSum - target, 0);
        preSum.put(currSum, preSum.getOrDefault(currSum, 0) + 1);
        
        res += helper(root.left, currSum, target) + helper(root.right, currSum, target);
        //注意回溯
        preSum.put(currSum, preSum.get(currSum) - 1);
        return res;
    }
```

