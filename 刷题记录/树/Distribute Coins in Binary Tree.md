# Distribute Coins in Binary Tree

### 问题

一颗二叉树的节点值表示这个节点有多少硬币，总共有n个节点，所有节点值的和为n。可以将一个节点的一个硬币移动到与它相邻的节点上。问最少需要移动多少次，使得每个节点有且只有一个硬币？

### 解决方案：递归，时间复杂度$O(n)$

首先，一个节点的所有子树都有硬币了，那么它把多余的肯定上交给父节点，即优先供给自己的子树，这样的移动次数才能最小。

因此维护硬币需要从下到上维护的。

从下到上维护节点，使得当前所有节点的硬币数为1，节点值可以为负数，表示需要多少硬币。每个节点都把`val-1`上交给父节点，同时`Math.abs(val-1)`也是该节点与父节点最少需要移动的次数。

```java
class Solution {
        public int distributeCoins(TreeNode root) {
                int[] ans=helpDistribute(root);
                return ans[0];
        }
    	//返回数组的第一个值表示移动的次数，第二个值表示当前节点的硬币数
        public int[] helpDistribute(TreeNode root){
                if(root==null) return new int[]{0, 1};
                int[] ans=new int[2];
                ans[1]=root.val;
                int[] l_info=helpDistribute(root.left);
                int[] r_info=helpDistribute(root.right);
                ans[1]+=l_info[1]+r_info[1]-2;
                ans[0]+=Math.abs(l_info[1]-1)+Math.abs(r_info[1]-1);
                ans[0]+=l_info[0]+r_info[0];
                return ans;
        }
}
```

