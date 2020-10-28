# Add Two Numbers II

### 问题

现有两个非空链表，表示两个整数，即每个node上的val代表该位的值，头结点为最高位。

要求不能进行翻转。

### 解决方案：时间复杂度$O(n)$，空间复杂度$O(n)$ 

逆序可以使用栈。

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        Stack<ListNode> stack1=new Stack<>();
        Stack<ListNode> stack2=new Stack<>();
        Stack<ListNode> stack3=new Stack<>();
        ListNode a=l1, b=l2;
        while(a!=null){
            stack1.push(a);
            a=a.next;
        }
        while(b!=null){
            stack2.push(b);
            b=b.next;
        }
        int carry=0;
        while(!stack1.isEmpty()&&!stack2.isEmpty()){
            ListNode first=stack1.pop();
            ListNode second=stack2.pop();
            int result=first.val+second.val+carry;
            carry=result/10;
            result=result%10;
            stack3.push(new ListNode(result));
        }
        Stack<ListNode> pt=stack1;
        if(stack1.size()<stack2.size()) pt=stack2;
        while(!pt.isEmpty()){
            ListNode cur=pt.pop();
            int result=cur.val+carry;
            carry=result/10;
            result=result%10;
            stack3.push(new ListNode(result));
        }
        if(carry!=0) stack3.push(new ListNode(1));
        ListNode root=stack3.pop();
        ListNode pre=root;
        while(!stack3.isEmpty()){
            ListNode cur=stack3.pop();
            pre.next=cur;
            pre=cur;
        }
        return root;
    }
}
```

