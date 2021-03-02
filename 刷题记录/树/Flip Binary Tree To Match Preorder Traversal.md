# Flip Binary Tree To Match Preorder Traversal

### 问题

一棵树`root`，一个前序序列`int[] voyage`。你可以将任意的结点的左右子树互换。问是否能够将`root`转化成前序序列为`voyage`的树。

### 解决方案：时间复杂度$O(n)$

#### 一、递归

一个helpFlip函数对应一个结点，一个结点对应一段`voyage`。

`helpFilp(root)=helpFlip(root.left)&&helpFlip(root.right)`。

当前结点的任务是检查当前结点的值是否与对应段的第一个值相同，检查左右两个子节点的值是否存在于对应段。

```java
class Solution {
        public List<Integer> flipMatchVoyage(TreeNode root, int[] voyage) {
                List<Integer> ans=new ArrayList<>();
                if(!helpFlip(root, voyage, 0, voyage.length-1, ans)){
                        ans.clear();
                        ans.add(-1);
                }
                return ans;
        }
        public boolean helpFlip(TreeNode root, int[] voyage, int l, int r, List<Integer> ans){
                if(root.val!=voyage[l]) return false;
                int left_index=-1, right_index=-1;
                if(root.left!=null){
                        left_index=findIndex(voyage, l, r, root.left.val);
                }
                if(root.right!=null){
                        right_index=findIndex(voyage, l, r, root.right.val);
                }
                if(root.left!=null&&left_index==-1) return false;
                if(root.right!=null&&right_index==-1) return false;
                int left_end=root.right!=null?right_index-1:r;
                int right_end=r;
                if(right_index!=-1&&left_index!=-1&&right_index<left_index) {
                        ans.add(root.val);
                        left_end=r;
                        right_end=left_index-1;
                }
                if(root.left!=null&&!helpFlip(root.left, voyage, left_index, left_end, ans)) 
                        return false;
                if(root.right!=null&&!helpFlip(root.right, voyage, right_index, right_end, ans)) 
                        return false;
                return true;
        }
        public int findIndex(int[] voyage, int l, int r, int key){
                for(int i=l; i<=r; i++){
                        if(voyage[i]==key) return i;
                }
                return -1;
        }
}
```

#### 二、DFS

注意前序序列的第一个总是根节点。左右子树互换只是选择dfs的方向罢了。

在外界维护一个`index`变量来指向前序的代搜索序号，dfs选择与`voyage[index]`相等的子节点的方向。当无法选择方向时或者当前搜索的结点的值与`voyage[index]`不同，则直接返回false。

```java
class Solution {
        int index=0;
        public List<Integer> flipMatchVoyage(TreeNode root, int[] voyage) {
                index=0;
                List<Integer> ans=new ArrayList<>();
                if(!dfs(root, voyage, ans)){
                        ans.clear();
                        ans.add(-1);
                }
                return ans;
        }
        public boolean dfs(TreeNode root, int[] voyage, List<Integer> ans){
                if(root==null) return true;
                if(root.val!=voyage[index]) return false;
                index++;
                if(root.left!=null&&root.left.val==voyage[index]){
                        if(!dfs(root.left, voyage, ans)||!dfs(root.right, voyage, ans)) return false;
                }else{
                        if(root.left!=null) ans.add(root.val);
                        if(!dfs(root.right, voyage, ans)||!dfs(root.left, voyage, ans)) return false;
                }
                return true;
        }
}
```

