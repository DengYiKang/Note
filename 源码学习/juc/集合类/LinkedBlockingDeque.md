# LinkedBlockingDeque

### 前言

LinkedBlockingDueue是一种有界阻塞队列，它的底层是双向链表，所以它是双向的。也就是说，在队首队尾都可以进行插入和删除操作。这样，LinkedBlockingDueue就支持FIFO（队列）、FILO（栈）两种操作方式。

### 成员

```java
static final class Node<E> {
    //被删除的节点的item域为null
    E item;

    Node<E> prev;

    Node<E> next;

    Node(E x) {
        item = x;
    }
    
}
```

![在这里插入图片描述](../../../pic/30)

>  注意，中间被删除的节点的前驱和后继两个成员没有改变，改变的只有旧前驱的`next`和旧后继的`pre`，使得链表上出发找不到被删除的节点，但是从被删除的节点出发仍能找到整个链表。这用于后面的迭代器，如果当前保存的节点被删除了，仍能找到下一个有效的节点。

```java
transient Node<E> first;//队首
transient Node<E> last;//队尾

/** 大小，节点的个数 */
private transient int count;

/** 容量 */
private final int capacity;

//与LinkedBlockingQueue不同的是，这只有一个锁
final ReentrantLock lock = new ReentrantLock();
private final Condition notEmpty = lock.newCondition();
private final Condition notFull = lock.newCondition();
```
### 构造器

```java
public LinkedBlockingDeque() {
    this(Integer.MAX_VALUE);
}

public LinkedBlockingDeque(int capacity) {
    if (capacity <= 0) throw new IllegalArgumentException();
    this.capacity = capacity;
}
```

注意，初始化时，队列中连dummy node都没有的。所以刚初始化的队列的`first`、`last`都为null。

### 入队操作

入队系列方法`add`、`put`、`offer`，实际都是在调用`linkFirst`或`linkLast`。仅以`putFirst`、`putLast`举例讲解。

#### putFirst

```java
//尝试失败则进入等待，响应中断
public void putFirst(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    Node<E> node = new Node<E>(e);
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        while (!linkFirst(node))
            notFull.await();
    } finally {
        lock.unlock();
    }
}
```
#### linkFirst
```java
//入队成功通知notEmpty，返回true，否则返回false
private boolean linkFirst(Node<E> node) {
    // assert lock.isHeldByCurrentThread();
    if (count >= capacity)
        return false;
    Node<E> f = first;
    node.next = f;
    first = node;
    if (last == null)
        last = node;
    else
        f.prev = node;
    ++count;
    notEmpty.signal();
    return true;
}
```

#### putLast

逻辑跟`putFirst`对称。

```java
public void putLast(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    Node<E> node = new Node<E>(e);
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        while (!linkLast(node))
            notFull.await();
    } finally {
        lock.unlock();
    }
}
```

### 出队操作

入队系列方法`remove`、`take`、`poll`，实际都是在调用`unlinkFirst`或`unlinkLast`。仅以`takeFirst`、`takeLast`举例讲解。

#### takeFirst

```java
//尝试失败进入等待，响应中断
public E takeFirst() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        E x;
        while ( (x = unlinkFirst()) == null)
            notEmpty.await();
        return x;
    } finally {
        lock.unlock();
    }
}
```
#### unlinkFirst
```java
//unlink成功通知notFull，返回对应的item，否则返回null
private E unlinkFirst() {
    // assert lock.isHeldByCurrentThread();
    Node<E> f = first;
    if (f == null)
        return null;
    Node<E> n = f.next;
    E item = f.item;
    f.item = null;
    f.next = f; // help GC
    first = n;
    if (n == null)
        last = null;
    else
        n.prev = null;
    --count;
    notFull.signal();
    return item;
}
```

#### takeLast

跟`takeFirst`完全对称。

```java
public E takeLast() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        E x;
        while ( (x = unlinkLast()) == null)
            notEmpty.await();
        return x;
    } finally {
        lock.unlock();
    }
}
```

### 删除内部节点

#### removeFirstOccurrence

从队首遍历到队尾，如果找到了节点，则执行`unlink`。

```java
public boolean removeFirstOccurrence(Object o) {
    if (o == null) return false;
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        for (Node<E> p = first; p != null; p = p.next) {
            if (o.equals(p.item)) {
                unlink(p);
                return true;
            }
        }
        return false;
    } finally {
        lock.unlock();
    }
}
```
#### unlink

注意，如果x是中间节点，会unlink使得从head或tail出发找不到x，但是没有改变x的前驱和后继，从x出发还是能找到链表上的所有节点的，这用于后面的迭代器。

```java
void unlink(Node<E> x) {
    // assert lock.isHeldByCurrentThread();
    Node<E> p = x.prev;
    Node<E> n = x.next;
    if (p == null) {
        unlinkFirst();
    } else if (n == null) {
        unlinkLast();
    } else {
        p.next = n;
        n.prev = p;
        x.item = null;
        // Don't mess with x's links.  They may still be in use by
        // an iterator.
        --count;
        notFull.signal();
    }
}
```
#### removeLastOccurrence

跟`removeFirstOccurrence`完全对称。

```java
public boolean removeLastOccurrence(Object o) {
    if (o == null) return false;
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        for (Node<E> p = last; p != null; p = p.prev) {
            if (o.equals(p.item)) {
                unlink(p);
                return true;
            }
        }
        return false;
    } finally {
        lock.unlock();
    }
}
```

### 迭代器

迭代器是弱一致性的，只有在初始化时会取获取锁。

```java
private abstract class AbstractItr implements Iterator<E> {
    
    Node<E> next;
    E nextItem;//item域提前保存

    private Node<E> lastRet;

    abstract Node<E> firstNode();
    abstract Node<E> nextNode(Node<E> n);

    AbstractItr() {
        //只有在初始化的时候才去获取锁
        final ReentrantLock lock = LinkedBlockingDeque.this.lock;
        lock.lock();
        try {
            next = firstNode();
            nextItem = (next == null) ? null : next.item;
        } finally {
            lock.unlock();
        }
    }

    private Node<E> succ(Node<E> n) {
        for (;;) {
            Node<E> s = nextNode(n);
            if (s == null)//表示到达末尾了
                return null;
            else if (s.item != null)//没有被删除
                return s;
            else if (s == n)//如果节点已经被unlink，则跳转到first
                //n.next=n，这种情况是被删除的第一个节点，自然是从头开始
                return firstNode();
            else//执行到这里，n有后继，但n的item为null，所以后移n，继续找
                n = s;
        }
    }

    void advance() {
        final ReentrantLock lock = LinkedBlockingDeque.this.lock;
        lock.lock();
        try {
            // assert next != null;
            next = succ(next);
            nextItem = (next == null) ? null : next.item;
        } finally {
            lock.unlock();
        }
    }

    public boolean hasNext() {
        return next != null;
    }

    public E next() {
        if (next == null)
            throw new NoSuchElementException();
        lastRet = next;
        E x = nextItem;
        advance();
        return x;
    }

    public void remove() {
        Node<E> n = lastRet;
        if (n == null)
            throw new IllegalStateException();
        lastRet = null;
        final ReentrantLock lock = LinkedBlockingDeque.this.lock;
        lock.lock();
        try {
            if (n.item != null)
                unlink(n);
        } finally {
            lock.unlock();
        }
    }
}
```

### 总结

+ 和ArrayBlockingQueue一样，使用一个Lock和两个Condition来控制并发和阻塞。因为两端都可以入队出队，所以用一个锁才能保证正确。
+ 和LinkedBlockingQueue不同的是，初始化时，连dummy node也没有。
+ 从first端移除的节点，next指针指向自身，以区别于first指向节点（next指针不会指向自身，可能为真实节点或null）。从last端移除的节点同理。
+ 中间的被删除节点，从该节点出发仍能找到链表上的所有节点，即该节点的`pre`与`next`没有改变，但不影响gc，因为没被引用。