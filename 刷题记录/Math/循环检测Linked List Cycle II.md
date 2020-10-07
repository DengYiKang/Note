# Linked List Cycle II

### 问题

输入一个链表，该链表仅含有一个环，输出环的起点，要求空间复杂度为$O(1)$

### 解决方案

#### Floyd's Tortoise and Hare (Cycle Detection)算法

现有两个指针first，second，first的步长为1，second的步长为2，它们相遇在点meet，meet离环的起点的距离为m。设链表的起点到环的起点的距离为s。

first走过的路程为$s+n_1r+m$，second走过路程为$2(s+n_1r+m)$，

那么有$2(s+n_1r+m)-(s+n_1r+m)=n_3r$，整理得$kr-m=s$。

现让first位于链表起点，second位于meet点，两者的步长为1，那么当first走的路程为s时，second走的路程为kr-m，两者恰好在环的起点相遇。

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode first=head, second=head;
        boolean isCircle=false;
        while(first!=null&&second!=null){
            first=first.next;
            if(second.next==null) return null;
            second=second.next.next;
            if(first==second){
                isCircle=true;
                break;
            }
        }
        if(!isCircle) return null;
        first=head;
        while(first!=second){
            first=first.next;
            second=second.next;
        }
        return first;
    }
}
```

