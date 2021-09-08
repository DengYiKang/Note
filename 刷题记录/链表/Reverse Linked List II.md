# Reverse Linked List II

给一个链表，转置第m到n个节点。

```java
public class Solution {
    public ListNode reverseBetween(ListNode head, int m, int n) {
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        //first part
        ListNode cur1 = dummy;
        ListNode pre1 = null;
        for(int i=0;i<m;i++){
            pre1 = cur1;
            cur1 = cur1.next;
        }
        
        //reverse
        ListNode cur2 = cur1;
        ListNode pre2 = pre1;
        ListNode q2;
        for(int i=m;i<=n;i++){
            q2 = cur2.next;
            cur2.next = pre2;
            pre2 = cur2;
            cur2 = q2;
        }
        
        //connect
        pre1.next = pre2;
        cur1.next = cur2;
        
        return dummy.next;
    }
}
```

以下是转置整个链表：

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        if(head==null) return head;
        ListNode dummy=new ListNode(0);
        dummy.next=head;
        ListNode pre=dummy, cur=head;
        while(cur!=null){
            ListNode tmp=cur.next;
            cur.next=pre;
            pre=cur;
            cur=tmp;
        }
        dummy.next=pre;
        head.next=null;
        return dummy.next;
    }
}
```

