# House Robber III

### 问题

现有一个二叉树，结点带有权值`val`，你可以从各个结点取出权值，但被取的结点不能相邻，求取出的权值的最大值。

### 解决方案：树上dp，时间复杂度$O(n)$，空间复杂度$O(logn)$

保存两种状态，`dpY`表示若当前结点取值且其作为root时的最大权值；`dpN`表示若当前结点不取值且作为root时的最大权值。

```java
class Solution {
    public int rob(TreeNode root) {
        Pair<Integer, Integer> pair=getVal(root);
        return Math.max(pair.getKey(), pair.getValue());
    }
    Pair<Integer, Integer> getVal(TreeNode root){
        if(root==null) return new Pair<Integer, Integer>(0, 0);
        Pair<Integer, Integer> l=getVal(root.left);
        Pair<Integer, Integer> r=getVal(root.right);
        int dpY=l.getValue()+r.getValue()+root.val;
        int dpN=Math.max(l.getKey(), l.getValue())+Math.max(r.getKey(), r.getValue());
        return new Pair<Integer, Integer>(dpY, dpN);
    }
}
```

