# Construct Binary Search Tree from Preorder Traversal

### 问题

给一个前序数组，要求构造出二叉搜索树。

### 解决方案：时间复杂度$O(n^2)$

```java
class Solution {
        public TreeNode bstFromPreorder(int[] preorder) {
                return build(preorder, 0, preorder.length-1);
        }
        public TreeNode build(int[] pre, int l, int r){
                if(r<l) return null;
                TreeNode root=new TreeNode(pre[l]);
                int mid=findIndex(pre, l, r);
                root.left=build(pre, l+1, mid-1);
                root.right=build(pre, mid, r);
                return root;
        }
        public int findIndex(int[] pre, int l, int r){
                int val=pre[l];
                for(int i=l+1; i<=r; i++){
                        if(pre[i]>val) return i;
                }
                return r+1;
        }
}
```

