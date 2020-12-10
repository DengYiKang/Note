# Find Duplicate Subtrees

### 问题

找到一个树的重复的子树，重复是指结构与值完全一致。返回重复的任意一个结点的引用。

### 解决方案：时间复杂度$O(n)$ 

一开始我是用先序中序后序中的任意两个来判断是否为重复的子树。然而发现如果一个树的各个节点的值相同时，这种方法无法判断。因此有必要把子结点为null的情况加入到判重的code中。

子节点如果为null的话返回`#`，然后总体用先序中序后序中的任意一种即可。

```java
class Solution {
    public List<TreeNode> findDuplicateSubtrees(TreeNode root) {
        HashMap<String, TreeNode> mp=new HashMap<>();
        List<TreeNode> duplicate=new ArrayList<>();
        traverse(root, mp, duplicate);
        return duplicate;
    }
    public String traverse(TreeNode node, HashMap<String, TreeNode> mp, List<TreeNode> duplicate){
        if(node==null) return "#";
        String left=traverse(node.left, mp, duplicate);
        String right=traverse(node.right, mp, duplicate);
        String code=node.val+","+left+","+right;
        if(mp.containsKey(code)&&!duplicate.contains(mp.get(code))){
            duplicate.add(mp.get(code));
        }
        mp.putIfAbsent(code, node);
        return code;
    }
}
```

