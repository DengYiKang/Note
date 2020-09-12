# Remove Duplicates from Sorted List II

### 问题

输入一个已排好序的链表，要求把重复的元素去除，比如1-1-2-3=>2-3，注意并不是去重。

### 解决方案

结点的拼接需要保存前一个结点，或者以p.next进行搜索。但若以p.next进行搜索，head本身无法被搜索到，因此最好定义一个头结点。

```java
//两次遍历
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if(head==null) return head;
        List<Integer> list=new ArrayList<>();
        ListNode node=new ListNode(-1, head);
        ListNode p=node.next, pre=node.next;
        while(p!=null){
            pre=p;
            //之前我在这搜索的是p.next，能用p本身尽量用吧，否则逻辑会变复杂
            while(p!=null&&p.val==pre.val) p=p.next;
            if(pre.next!=p) list.add(pre.val);
        }
        p=node;
        int tail=0;
        while(p.next!=null&&tail<list.size()){
            if(p.next.val==list.get(tail)){
                p.next=p.next.next;
            }
            else if(list.get(tail)<p.next.val) tail++;
            else{
                p=p.next;
            }
        }
        return node.next;
    }
}
```

```java
//一次遍历
public ListNode deleteDuplicates(ListNode head) {
        if(head==null) return null;
        ListNode FakeHead=new ListNode(0);
        FakeHead.next=head;
        ListNode pre=FakeHead;
        ListNode cur=head;
        while(cur!=null){
            while(cur.next!=null&&cur.val==cur.next.val){
                cur=cur.next;
            }
            if(pre.next==cur){
                pre=pre.next;
            }
            else{
                pre.next=cur.next;
            }
            cur=cur.next;
        }
        return FakeHead.next;
    }
```

