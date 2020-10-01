# Odd Even Linked List

### 问题

现有一个链表，把奇数结点排在前面，偶数结点排在后面

### 解决方案

```java
class Solution {
    public ListNode oddEvenList(ListNode head) {
        if(head==null||head.next==null||head.next.next==null) return head;
        ListNode evenHead=head, oddHead=head.next;
        ListNode p=evenHead, q=oddHead;
        while(p.next!=null&&q.next!=null){
            if(p.next==q){
                p.next=q.next;
                p=p.next;
                //注意这个else，每次循环只对其中一个指针变换
            }else if(q.next==p){
                q.next=p.next;
                q=q.next;
            }
        }
        //注意，否则会生成环
        if(p.next==q) p.next=null;
        if(q.next==p) q.next=null;
        p.next=oddHead;
        return evenHead;
    }
}
```

