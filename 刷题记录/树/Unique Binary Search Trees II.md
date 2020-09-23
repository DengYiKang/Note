# Unique Binary Search Trees II

### 问题

输入整数`n`，生成存储`1~n`的BST所有异构体

### 解决方案

```java
class Solution {
    
    public List<TreeNode> generateTrees(int n) {
        if(n==0) return new ArrayList<>();
        return build(1, n);
    }
    List<TreeNode> build(int lo, int hi){
        List<TreeNode> trees=new ArrayList<>();
        if(lo>hi){
            trees.add(null);
            return trees;
        }
        for(int mid=lo; mid<=hi; mid++){
            List<TreeNode> left=build(lo, mid-1);
            List<TreeNode> right=build(mid+1, hi);
            for(int i=0; i<left.size(); i++){
                for(int j=0; j<right.size(); j++){
                    TreeNode root=new TreeNode(mid);
                    root.left=left.get(i);
                    root.right=right.get(j);
                    trees.add(root);
                }
            }
        }
        return trees;
    }
}
```

