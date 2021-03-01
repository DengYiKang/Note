# Check Completeness of a Binary Tree

### 问题

检查是否为完全二叉树

### 解决方案：时间复杂度$O(n)$，空间复杂度$O(logn)$

bfs。维护一个标志`flag`，如果某一结点的左子树或右子树为空，那么`flag`变为true。如果某一结点存在子树且`flag`为true时，那么该树不是完全二叉树。

```java
class Solution {
        public boolean isCompleteTree(TreeNode root) {
                Queue<TreeNode> queue=new LinkedList<>();
                queue.offer(root);
                boolean flag=false;
                while(!queue.isEmpty()){
                        int size=queue.size();
                        for(int i=0; i<size; i++){
                                TreeNode cur=queue.poll();
                                if(cur.left!=null) {
                                        if(flag) return false;
                                        queue.offer(cur.left);
                                }
                                else flag=true;
                                if(cur.right!=null) {
                                        if(flag) return false;
                                        queue.offer(cur.right);
                                }
                                else flag=true;
                        }
                }
                return true;
        }
}
```

