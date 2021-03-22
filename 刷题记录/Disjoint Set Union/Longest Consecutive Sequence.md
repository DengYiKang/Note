# Longest Consecutive Sequence

### 问题

乱序整型数组`nums`，求最长的连续序列的长度。

**Example 1:**

```
Input: nums = [100,4,200,1,3,2]
Output: 4
Explanation: The longest consecutive elements sequence is [1, 2, 3, 4]. Therefore its length is 4.
```

### 解决方案：并查集，时间复杂度$O(n)$，空间复杂度$O(n)$

用map来保存int->node的映射，对当前数，从map中取出左右相邻的数作连接操作，优先以左边的数为根。每个node保存当前集合的元素的数量。对于root，用一个list来存。我们把所有连接操作的左边的数统统加入到list，因为只需要求最大值，因此即使list中有重复的node也没什么关系。

```java
class Solution {
    public int longestConsecutive(int[] nums) {
        List<Node> roots=new ArrayList<>();
        HashMap<Integer, Node> mp=new HashMap<>();
        for(int x:nums){
            if(mp.containsKey(x)) continue;
            Node node=new Node(x);
            Node left=mp.getOrDefault(x-1, null);
            Node right=mp.getOrDefault(x+1, null);
            if(right!=null) join(node, right);
            if(left!=null) join(left, node);
            else roots.add(node);
            mp.put(x, node);
        }
        int ans=0;
        for(Node node:roots){
            ans=Math.max(ans, node.cnt);
        }
        return ans;
    }
    class Node{
        int val;
        Node parent;
        int cnt;
        Node(int val){
            this.val=val;
            this.cnt=1;
            this.parent=this;
        }
    }
    private Node findRoot(Node u){
        if(u.parent==u) return u;
        Node root=u;
        while(root.parent!=root) root=root.parent;
        Node tmp=null;
        while(u!=root){
            tmp=u.parent;
            u.parent=root;
            u=tmp;
        }
        return root;
    }
    private void join(Node u, Node v){
        Node root_u=findRoot(u);
        Node root_v=findRoot(v);
        if(root_u!=root_v){
            root_v.parent=root_u;
            root_u.cnt+=root_v.cnt;
        }
    }
}
```

