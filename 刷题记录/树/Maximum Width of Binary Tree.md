# Maximum Width of Binary Tree

### 问题

求一个树的最大宽度，该宽度是指单层宽度，注意两节点之间包括了null结点，它们也计算在内。

### 解决方案：时间复杂度$O(n)$

涉及到每层的问题，考虑用队列。但是需要考虑null结点，因此需要一个宽度计数。

现以距离最左端点的距离计数（不管是存在的结点还是null结点），求某层的宽度时只需要把该层最右的非null结点和最左端的非null结点的计数相减即可。

考虑到`node`的计数为`pre`，则`node.left`的计数为`pre*2`，`node.right`的计数为`pre*2+1`。

```java
class Solution {
    private class Node{
        TreeNode node;
        int width;
        Node(TreeNode node, int width){
            this.node=node;
            this.width=width;
        }
    }
    public int widthOfBinaryTree(TreeNode root) {
        if(root==null) return 0;
        int max_width=-1;
        Queue<Node> queue=new LinkedList<>();
        queue.offer(new Node(root, 0));
        while(!queue.isEmpty()){
            int len=queue.size();
            int left_width=0;
            int right_width=0;
            for(int i=0; i<len; i++){
                Node cur=queue.poll();
                if(i==0) left_width=cur.width;
                if(i==len-1) right_width=cur.width;
                if(cur.node.left!=null){
                    queue.offer(new Node(cur.node.left, cur.width*2));
                }
                if(cur.node.right!=null){
                    queue.offer(new Node(cur.node.right,
                                         cur.width*2+1));
                }
            }
            max_width=Math.max(max_width, right_width-left_width+1);
        }
        return max_width;
    }
    
}
```

