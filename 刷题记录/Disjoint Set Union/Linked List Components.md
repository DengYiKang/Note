# Linked List Components

### 问题

有一个链表，长度为`n`，链上为不重复的`0～n-1`的数。现给一组数`G[1...n]`，`G`为链表上数值的子集，在链表上相邻的数之间可以组成一个块（`a1, a2`也能和`a3`组成一个块）。求问`G`的最小块数。

### 解决方案：dsu，时间复杂度$O(n)$

可以用并查集的root来代表一个块，在uinon之后，检查root的个数有多少即可。

```java
class Solution {
    public int numComponents(ListNode head, int[] G) {
        HashSet<Integer> gset=new HashSet<>();
        for(int i=0; i<G.length; i++){
            gset.add(G[i]);
        }
        int[] parent=new int[10001];
        for(int i=0; i<parent.length; i++){
            parent[i]=i;
        }
        int pre=-2;
        ListNode cur=head;
        while(cur!=null){
            if(gset.contains(cur.val)&&gset.contains(pre)){
                union(pre, cur.val, parent);
            }
            pre=cur.val;
            cur=cur.next;
        }
        HashSet<Integer> root_set=new HashSet<>();
        for(int i=0; i<G.length; i++){
            root_set.add(find_root(G[i], parent));
        }
        return root_set.size();
    }
    public int find_root(int u, int[] parent){
        if(u==parent[u]) return u;
        int root=u;
        while(root!=parent[root]) root=parent[root];
        while(u!=parent[u]){
            int tmp=parent[u];
            parent[u]=root;
            u=tmp;
        }
        return root;
    }
    public void union(int u, int v, int[] parent){
        int root_u=find_root(u, parent);
        int root_v=find_root(v, parent);
        if(root_u!=root_v){
            parent[root_u]=root_v;
        }
    }
}
```

### 解决方案：根据union的次数来计算块

```java
class Solution {
    public int numComponents(ListNode head, int[] nums) {
        HashSet<Integer> set=new HashSet<>();
        for(int i=0; i<nums.length; i++){
            set.add(nums[i]);
        }
        int ans=set.size();
        while(head!=null&&head.next!=null){
            if(set.contains(head.val)&&set.contains(head.next.val)){
                ans--;
            }
            head=head.next;
        }
        return ans;
    }
}
```

