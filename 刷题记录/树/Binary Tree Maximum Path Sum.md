# Binary Tree Maximum Path Sum

### 问题

找到树中的一个路径使得该路径上的值的总和最大。

### 解决方案：dp，时间复杂度$O(n)$

`getLen(node)`表示以`node`为起点，向左或向右最大的路径值，如果小于0，直接返回0，表示不取当前`node`。

那么对树bfs，对每个node，`ans=max(getLen(node.left)+getLen(node.right)+node.val)`。

```java
class Solution {
        HashMap<TreeNode, Integer> mp;
        public int maxPathSum(TreeNode root) {
                mp=new HashMap<>();
                Queue<TreeNode> queue=new LinkedList<>();
                queue.offer(root);
                int ans=Integer.MIN_VALUE;
                while(!queue.isEmpty()){
                        TreeNode node=queue.poll();
                        ans=Math.max(ans, getLen(node.left)+getLen(node.right)+node.val);
                        if(node.left!=null) queue.offer(node.left);
                        if(node.right!=null) queue.offer(node.right);
                }
                return ans;
        }
        public int getLen(TreeNode node){
                if(node==null) return 0;
                if(mp.containsKey(node)) return mp.get(node);
                int ans=Math.max(getLen(node.left), getLen(node.right))+node.val;
                ans=Math.max(ans, 0);
                mp.put(node, ans);
                return ans;
        }
}
```

