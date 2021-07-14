# Sort List

### 问题

输入一个链表，对它进行排序，要求时间复杂度为$O(nlogn)$

### 解决方案

归并排序

```java
class Solution {
    public ListNode sortList(ListNode head) {
        if(head==null||head.next==null) return head;
        ListNode slow=head, fast=head, pre=slow;
        while(fast!=null&&fast.next!=null){
            pre=slow;
            slow=slow.next;
            fast=fast.next.next;
        }
        pre.next=null;
        ListNode a=sortList(head);
        ListNode b=sortList(slow);
        return merge(a, b);
    }
    
    ListNode merge(ListNode a, ListNode b){
        ListNode fake=new ListNode(), p=fake;
        while(a!=null&&b!=null){
            if(a.val<b.val){
                p.next=a;
                p=p.next;
                a=a.next;
            }else{
                p.next=b;
                p=p.next;
                b=b.next;
            }
        }
        if(a!=null) p.next=a;
        if(b!=null) p.next=b;
        return fake.next;
    }
}
```

找中间值这样比较好，low.next开始的长度最多为n/2。

```java
    while(fast.next!=null){
        fast=fast.next;
        if(fast.next!=null) fast=fast.next;
        else break;
        low=low.next;
    }
```
