# Delete Node in a Linked List

### 问题

单向链表，给一个需要删除的node，要求在该链表上删除他，这个node不会是tail。

### 解决方案：交换节点

```java
class Solution {
    public void deleteNode(ListNode node) {
        node.val=node.next.val;
        node.next=node.next.next;
    }
}
```

