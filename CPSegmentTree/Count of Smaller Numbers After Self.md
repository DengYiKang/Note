# Count of Smaller Numbers After Self

### 问题

现有一个整型数组`nums`，求一个`count`数组，`count[i]`表示`i+1~n`的位置上有多少个比`nums[i]`小的数。

```
Input: nums = [5,2,6,1]
Output: [2,1,1,0]
Explanation:
To the right of 5 there are 2 smaller elements (2 and 1).
To the right of 2 there is only 1 smaller element (1).
To the right of 6 there is 1 smaller element (1).
To the right of 1 there is 0 smaller element.
```

### 解决方案：线段树，时间复杂度$O(nlogn)$

使用线段树，同时将区间点压缩到连续的数。

```java
class Solution {
    class SegmentTree{
        int lo;
        int hi;
        int cnt;
        SegmentTree left;
        SegmentTree right;
        int mid(){
            return (lo+hi)>>1;
        }
        SegmentTree(int lo, int hi){
            this.lo=lo;
            this.hi=hi;
            this.cnt=0;
        }
    }
    void add(SegmentTree node, int x){
        if(x<node.lo||x>node.hi) return;
        node.cnt++;
        if(node.lo==node.hi){
            return;
        }
        int mid=node.mid();
        if(x<=mid){
            if(node.left==null) node.left=new SegmentTree(node.lo, mid);
            add(node.left, x);
        }else{
            if(node.right==null) node.right=new SegmentTree(mid+1, node.hi);
            add(node.right, x);
        }
    }
    int query(SegmentTree node, int x){
        if(node==null) return 0;
        if(x>node.hi) return node.cnt;
        if(x<node.lo) return 0;
        int mid=node.mid();
        if(x<=mid) return query(node.left, x);
        else return query(node.left, x)+query(node.right, x);
    }
    public List<Integer> countSmaller(int[] nums) {
        Map<Integer, Integer> x2i=new HashMap<>();
        List<Integer> list=Arrays.stream(nums).boxed().collect(Collectors.toList());
        list=new ArrayList<Integer>(new HashSet<Integer>(list));
        Collections.sort(list);
        int tail=0;
        for(int x:list){
            x2i.put(x, tail++);
        }
        SegmentTree root=new SegmentTree(0, tail-1);
        List<Integer> ans=new ArrayList<>();
        for(int i=nums.length-1; i>=0; i--){
            ans.add(query(root, x2i.get(nums[i])));
            add(root, x2i.get(nums[i]));
        }
        Collections.reverse(ans);
        return ans;
    }
}
```

