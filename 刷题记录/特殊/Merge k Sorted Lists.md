# Merge k Sorted Lists

### 问题

有k个升序链表，要求合并成一个升序链表。

### 解决方案：时间复杂度$O(nlgk)$

因为k个链表都是升序的，因此最小值存在与链表的第一个节点所组成的集合。

维护一个最小堆，这个堆有每个链表的第一个节点组成，当取出一个最小值，该最小值所在的链表再取出剩下的第一个节点加入堆中，不断重复。

```java
class Solution {
        class Node{
                ListNode node;
                int pos;
                Node(ListNode node, int pos){
                        this.node=node;
                        this.pos=pos;
                }
        }
        public ListNode mergeKLists(ListNode[] lists) {
                PriorityQueue<Node> pq=new PriorityQueue<>((a, b)->a.node.val-b.node.val);
                for(int i=0; i<lists.length; i++){
                        if(lists[i]==null) continue;
                        pq.offer(new Node(lists[i], i));
                        lists[i]=lists[i].next;
                }
                ListNode head=null, tail=null;
                while(!pq.isEmpty()){
                        Node cur=pq.poll();
                        if(head==null){
                                head=cur.node;
                                tail=head;
                        }else{
                                tail.next=cur.node;
                                tail=tail.next;
                        }
                        if(lists[cur.pos]!=null) {
                                pq.offer(new Node(lists[cur.pos], cur.pos));
                                lists[cur.pos]=lists[cur.pos].next;
                        }
                }
                return head;
        }
}
```

