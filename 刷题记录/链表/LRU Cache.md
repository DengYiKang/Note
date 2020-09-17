# LRU Cache

### 问题

要求实现一个LRU缓存，get与put时间复杂度为$O(1)$

### 解决方案

使用链表存储数据，最新使用的放在最前面，使用hashmap来映射key与结点。

把各种原子操作独立出来会比较好写。

```shell
class LRUCache {
    
    Node head, tail;
    int cnt, capacity;
    HashMap<Integer, Node> mp;

    class Node{
        int key;
        int val;
        Node pre;
        Node next;
        public Node(){}
        public Node(int key, int val){
            this.key=key;
            this.val=val;
        }
    }
    
    public LRUCache(int capacity) {
        this.capacity=capacity;
        mp=new HashMap<>();
        head=new Node();
        tail=new Node();
        head.next=tail;
        head.pre=tail;
        tail.next=head;
        tail.pre=head;
        cnt=0;
    }
    
    public void addNode(Node node){
        node.next=head.next;
        head.next=node;
        node.pre=head;
        node.next.pre=node;
        mp.put(node.key, node);
        cnt++;
    }
    
    public void dropNode(Node node){
        Node pre=node.pre;
        Node next=node.next;
        pre.next=next;
        next.pre=pre;
        mp.remove(node.key);
        cnt--;
    }
    public void removeToFirst(Node node){
        dropNode(node);
        addNode(node);
    }
    
    public int get(int key) {
        if(!mp.containsKey(key)) return -1;
        Node node=mp.get(key);
        removeToFirst(node);
        return node.val;
    }
    
    public void put(int key, int val) {
        if(mp.containsKey(key)){
            Node node=mp.get(key);
            removeToFirst(node);
            node.val=val;
        }else{
            if(cnt==capacity){
                dropNode(tail.pre);
                addNode(new Node(key, val));
            }else{
                addNode(new Node(key, val));
            }
        }
    }
}
```

