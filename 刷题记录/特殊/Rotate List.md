# Rotate List

### 问题

输入一个链表，一个整数k，输出将链表的末端接至首部操作k次后的链表

```java
Input: 0->1->2->NULL, k = 4
Output: 2->0->1->NULL
Explanation:
rotate 1 steps to the right: 2->0->1->NULL
rotate 2 steps to the right: 1->2->0->NULL
rotate 3 steps to the right: 0->1->2->NULL
rotate 4 steps to the right: 2->0->1->NULL
```

### 解决方案

如何找到末k个结点？可先由指针p指向第k个结点，q指向head，两指针同时移动，当p指向尾部时，q就指向末k个结点了。

令k'=k%length，length为链表长度，可以注意到把末k'个子节点按原顺序放到首部就可以了。

注意各种边界条件，最好先大致写好，再按样例调边界。

```java
class Solution {
    public ListNode rotateRight(ListNode head, int k) {
        if(head==null) return null;
        ListNode p=head, q=head, ans;
        int len=-1;
        for(int i=0; i<k; i++){
            p=p.next;
            if(p==null) {
                len=i+1;
                break;
            }
        }
        if(len!=-1){
            k=k%len;
            p=head;
            for(int i=0; i<k; i++) p=p.next;
        }
        while(p.next!=null){
            p=p.next;
            q=q.next;
        }
        p.next=head;
        ans=q.next;
        q.next=null;
        return ans;
    }
}
```

