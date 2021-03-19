# Symmetric Tree

### 问题

判断一个树是否是轴对称的。

### 解决方案：递归，时间复杂度$O(n)$

这也可以用bfs来做，判断每一层是否是轴对称的，如果某个子树为null，那就用一个标识符代表入队。

这里用递归做。可以将问题转化为“判断两个树是否是翻转的”。

```java
class Solution {
        public boolean isSymmetric(TreeNode root) {
                if(root==null) return true;
                return flip(root.left, root.right);
        }
        public boolean flip(TreeNode u, TreeNode v){
                if(u==null&&v==null) return true;
                if(u!=null&&v!=null){
                        if(u.val!=v.val) return false;
                        return flip(u.left, v.right)&&flip(u.right, v.left);
                }
                return false;
        }
}
```

