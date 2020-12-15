## Longest Univalue Path

### 问题

现有一棵树，找到这棵树中最长的路径，路径可以不包括根节点。

![img](../pic/13.jpg)

### 解决方案：

#### 一、遍历+递归，时间复杂度：$worst\;case\;O(n^2)$

用bfs遍历，对遍历的当前结点`cur`计算包含它在内的最长的路径，注意`cur`计算两子节点的单向链。

```java
class Solution {
    public int longestUnivaluePath(TreeNode root) {
        if(root==null) return 0;
        HashSet<TreeNode> vis=new HashSet<>();
        Queue<TreeNode> queue=new LinkedList<>();
        int ans=0;
        queue.offer(root);
        while(!queue.isEmpty()){
            TreeNode cur=queue.poll();
            ans=Math.max(getLUP(cur, cur.val, vis), ans);
            if(cur.left!=null) queue.offer(cur.left);
            if(cur.right!=null) queue.offer(cur.right);
        }
        return ans-1;
    }
    public int getLUP(TreeNode root, int val, HashSet<TreeNode> vis){
        if(root==null||root.val!=val||vis.contains(root)) return 0;
        int l_cnt=getPath(root.left, val);
        int r_cnt=getPath(root.right, val);
        int ans=l_cnt+ r_cnt+1;
        vis.add(root);
        return ans;
    }
    //单向链
    public int getPath(TreeNode root, int val){
        if(root==null||root.val!=val) return 0;
        int l_cnt=getPath(root.left, val);
        int r_cnt=getPath(root.right, val);
        int ans=Math.max(l_cnt, r_cnt)+1;
        return ans;
    }
}
```

#### 二、递归，时间复杂度$O(n)$ 

前面一直有种思维定式，觉得当前递归的函数的功能对应哪种值就维护哪种值（比如计算最大值，就维护遍历中得到的最大值）。

因为该问题有两种情况，如上个方法中把功能分为两种`getLUP`和`getPath`，前者是作为中间结点的情况，后者是作为端点的情况（即单向）。

但是我们仍可以只实现单向的功能，而在实现这个功能的同时可以得到两边的单向长度，因此间接地得到了`LUP`的长度，因此维护一个最大值就好了。

```java
class Solution {
    int ans;
    public int longestUnivaluePath(TreeNode root) {
        if(root==null) return 0;
        ans=0;
        getPath(root);
        return ans-1;
    }
    //注意val值不能在外边判断（即直接返回，不然无法递归到所有结点）
    public int getPath(TreeNode root){
        if(root==null) return 0;
        int l_cnt=getPath(root.left);
        int r_cnt=getPath(root.right);
        int left=0, right=0;
        if(root.left!=null&&root.left.val==root.val){
            left=l_cnt;
        }
        if(root.right!=null&&root.right.val==root.val){
            right=r_cnt;
        }
        ans=Math.max(ans, left+right+1);
        return Math.max(left, right)+1;
    }
}
```

