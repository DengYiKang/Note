# Palindrome Linked List

### 问题

给一个链表，要求判断他是不是回文的。要求时间复杂度$O(n)$，空间复杂度$O(1)$。

### 解决方案

找到中点位置，然后将中点位置后的链表进行翻转，这个中点位置作为左右两边检验的停止标记。

```java
class Solution {
    public boolean isPalindrome(ListNode head) {
        ListNode slow=head;
        ListNode fast=head;
        while(fast.next!=null&&fast.next.next!=null){
            fast=fast.next.next;
            slow=slow.next;
        }
        if(fast.next!=null) fast=fast.next;
        if(fast==slow) return true;
        reverse(slow);
        ListNode cur=head;
        while(cur!=slow&&cur.val==fast.val){
            cur=cur.next;
            fast=fast.next;
        }
        return cur.val==fast.val;
    }
    void reverse(ListNode head){
        ListNode pre=head;
        ListNode cur=head.next;
        while(cur!=null){
            ListNode tmp=cur.next;
            cur.next=pre;
            pre=cur;
            cur=tmp;
        }
    }
}
```

