# Convert Sorted List to Binary Search Tree

### 问题

输入一个升序的链表，要求输出一个AVL

### 解决方案

注意该链表是排好序的，直接二等分构造即可，不需要AVL模板（AVL模板适合乱序的序列）

```java
class Solution {
    
    List<Integer> nums;
    public TreeNode sortedListToBST(ListNode head) {
        if(head==null) return null;
        nums=new ArrayList<>();
        while(head!=null){
            nums.add(head.val);
            head=head.next;
        }
        return build(0, nums.size()-1);
    }
    TreeNode build(int left, int right){
        if(left>right) return null;
        int mid=(left+right)/2;
        TreeNode node=new TreeNode(nums.get(mid));
        node.left=build(left, mid-1);
        node.right=build(mid+1, right);
        return node;
    }
}
```

