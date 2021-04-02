# Sliding Window Maximum

### 问题

一个整型数组`nums`，长度为k的滑动窗口，滑动窗口每次往右移动一格，求滑动窗口中最大数组成的数组。

```
Input: nums = [1,3,-1,-3,5,3,6,7], k = 3
Output: [3,3,5,5,6,7]
Explanation: 
Window position                Max
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

### 解决方案：线段树，时间复杂度$O(nlogn)$

#### 链表版本

```java
class Solution {
    class SegmentTree{
        int val;
        int lo;
        int hi;
        SegmentTree left;
        SegmentTree right;
        int mid(){
            return (lo+hi)>>1;
        }
        SegmentTree(int lo, int hi){
            this.lo=lo;
            this.hi=hi;
            this.val=Integer.MIN_VALUE;
        }
    }
    SegmentTree build(int[] nums, int l, int r){
        if(l>r) return null;
        SegmentTree node=new SegmentTree(l, r);
        if(l==r) node.val=nums[l];
        else{
            int mid=(l+r)>>1;
            node.left=build(nums, l, mid);
            node.right=build(nums, mid+1, r);
            node.val=Math.max(node.left!=null?node.left.val:Integer.MIN_VALUE,
                             node.right!=null?node.right.val:Integer.MIN_VALUE);
        }
        return node;
    }
    int query(SegmentTree node, int l, int r){
        if(l>r) return Integer.MIN_VALUE;
        if(l==node.lo&&r==node.hi) return node.val;
        int mid=node.mid();
        return Math.max(query(node.left, l, Math.min(mid, r)), 
                        query(node.right, Math.max(mid+1, l), r));
    }
    public int[] maxSlidingWindow(int[] nums, int k) {
        SegmentTree root=build(nums, 0, nums.length-1);
        int[] ans=new int[nums.length-k+1];
        for(int i=0; i+k-1<nums.length; i++){
            ans[i]=query(root, i, i+k-1);
        }
        return ans;
    }
}
```

#### 数组版本

```java
class Solution {
    class SegmentTree{
        int[] t;
        SegmentTree(int size){
            t=new int[size<<2];
        }
        void build(int u, int tl, int tr, int[] nums){
            if(tl==tr){
                t[u]=nums[tr-1];
            }else{
                int mid=(tl+tr)>>1;
                build(u<<1, tl, mid, nums);
                build((u<<1)+1, mid+1, tr, nums);
                t[u]=Math.max(t[u<<1], t[(u<<1)+1]);
            }
        }
        int query(int u, int tl, int tr, int l, int r){
            if(l>r) return Integer.MIN_VALUE;
            if(tl==l&&tr==r) return t[u];
            int mid=(tl+tr)>>1;
            int l_val=query(u<<1, tl, mid, l, Math.min(mid, r));
            int r_val=query((u<<1)+1, mid+1, tr, Math.max(l, mid+1), r);
            return Math.max(l_val, r_val);
        }
    }
    public int[] maxSlidingWindow(int[] nums, int k) {
        SegmentTree st=new SegmentTree(nums.length);
        st.build(1, 1, nums.length, nums);
        int[] ans=new int[nums.length-k+1];
        for(int lo=0; lo+k-1<nums.length; lo++){
            ans[lo]=st.query(1, 1, nums.length, lo+1, lo+k);
        }
        return ans;
    }
}
```

