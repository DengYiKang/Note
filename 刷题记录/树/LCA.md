# 最近公共祖先（LCA）

### 以链表方式存储

#### 递归：时间复杂度$O(n)$，空间复杂度$O(n)$ 

在递归的过程中，若发现子树存在查询结点中的某一个，那么设置标志为1，当递归位置处于点node时，若左右子树均有标记或node点与左右子树之一有标记，那么node点即为LCA。

```java
class Solution {
    
    TreeNode ans;
    
    boolean findOne(TreeNode node, TreeNode p, TreeNode q){
        if(node==null) return false;
        int left=findOne(node.left, p, q)?1:0;
        int right=findOne(node.right, p, q)?1:0;
        int cur=(node==p||node==q)?1:0;
        if(left+cur+right>=2) this.ans=node;
        return (left+cur+right>0)
    }
    
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        ans=null;
        findOne(root, p, q);
        return ans;
    }
}
```

### 以数组形式存储

#### Tarjan离线算法

输入一串查询，输出对应的结果，时间复杂度为`O(n+m)`，其中`n`为结点数，`m`为查询数。

下面为伪代码：

```python
#merge与find均为DSU的方法
def dfs(u):
    for each edge (u, v):
        dfs(v)
        #注意要合并到u上
        merge(u,v)
        visited[v]=true
	end for
    for each query (u, e):
        if e is visted:
            lca(u,e)=find(e)
        end if
    end for
```

