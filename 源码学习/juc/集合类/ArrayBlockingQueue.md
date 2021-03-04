# ArrayBlockingQueue

### 前言

`ArrayBlockingQueue`是一种FIFO（first-in-first-out 先入先出）的有界阻塞队列，底层是循环数组，也支持从内部删除元素。并发操作依赖于加锁的控制，支持阻塞式的入队出队操作。正因为有界，所以才会阻塞。

加锁实现完全依赖于AQS。

### 成员

```java
//保存队列元素的数组
final Object[] items;

//下次出队的位置
int takeIndex;

//下次入队的位置
int putIndex;

//队列中元素的数量
int count;

final ReentrantLock lock;
private final Condition notEmpty;
private final Condition notFull;
```
队列中非null元素的范围是`[takeIndex, putIndex)`的左闭右开的区间。考虑到底层是循环数组，有可能`putIndex`比`takeIndex`小。二者相等也很好理解，代表队列中每个元素都是非null元素。

### 构造器

```java
public ArrayBlockingQueue(int capacity) {
    this(capacity, false);
}

public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0)
        throw new IllegalArgumentException();
    this.items = new Object[capacity];
    lock = new ReentrantLock(fair);
    notEmpty = lock.newCondition();
    notFull =  lock.newCondition();
}
```
构造器默认使用的是非公平的ReentrantLock，当然你也可以指定为公平的ReentrantLock。

```java
public ArrayBlockingQueue(int capacity, boolean fair, Collection<? extends E> c) {
    this(capacity, fair);
    final ReentrantLock lock = this.lock;
    lock.lock(); // 加锁只是为了保证可见性
    try {
        int i = 0;
        try {
            for (E e : c) {
                checkNotNull(e);
                items[i++] = e;//如果传入集合的个数超过了容量，抛出异常被catch，最多放capacity个
            }
        } catch (ArrayIndexOutOfBoundsException ex) {
            throw new IllegalArgumentException();
        }
        count = i;//循环结束，i刚好是放置的个数
        putIndex = (i == capacity) ? 0 : i;//循环结束，i也刚好是最后放置元素的索引+1
    } finally {
        lock.unlock();
    }
}
```
如果传入集合的个数超过了容量，抛出异常被catch，最多放capacity个元素。

### 入队

#### add

```java
//ArrayBlockingQueue.java
    public boolean add(E e) {
        return super.add(e);
    }
    
//AbstractQueue.java
    public boolean add(E e) {
        if (offer(e))
            return true;
        else//返回false的处理不一样
            throw new IllegalStateException("Queue full");
    }
    
//Queue.java(接口文件)
	boolean offer(E e);
```

`add`的实现是依靠父类的`add`实现，后者又依靠于子类的`offer`实现。所以，`add`就是在调用自己的`offer`方法。

#### offer

```java
//跟put不同的是，offer只尝试一次，如果不符合条件直接退出，且不响应中断
public boolean offer(E e) {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        if (count == items.length)
            return false;
        else {
            enqueue(e);
            return true;
        }
    } finally {
        lock.unlock();
    }
}
```

- 入队是一个写操作，自然需要加锁。`lock.lock()`不响应中断，线程会一直阻塞直到抢到锁。
- 队列已满，则无法入队，返回false。
- 队列未满，则可以入队，返回true。

```java
 private void enqueue(E x) {
        // assert lock.getHoldCount() == 1; 即保证，调用此函数的线程已经获得锁
        // assert items[putIndex] == null;  即保证，入队位置是空的
        final Object[] items = this.items;
        items[putIndex] = x;
        if (++putIndex == items.length)//更新putIndex
            putIndex = 0;
        count++;//大小加1
        notEmpty.signal();//通知阻塞在notEmpty条件队列上的第一个线程
    }
```

- 在putIndex位置是空的，我们直接往putIndex索引上入队。
- 右移putIndex，按照循环数组的方式。
- 队列大小加1。
- 通知阻塞在notEmpty条件队列上的线程，只通知一个线程。

#### put

```java
//与offer不同的是，当满额无法插入时将会被阻塞，只有当有位置时才会被signal，并且响应中断(await响应)
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length)
            notFull.await();
        enqueue(e);
    } finally {
        lock.unlock();
    }
}
```

+ 在进入加锁代码之前，执行的是`lock.lockInterruptibly()`。这意味着，当前线程在抢到锁之前，如果被中断了，put方法会抛出中断异常。
+ 进入加锁代码之后，当前线程便已是获得了锁。但获得了锁，和队列当前是空是满根本没有关系。
+ 如果队列未满，那么不会执行`notFull.await()`，直接入队。
+ 需要使用`while (count == items.length)`来防止虚假唤醒，即使当前线程从`notFull.await()`恢复执行了，如果当前队列还是满的，那么应该重新进入条件队列。所以，需要重新检查一遍`count == items.length`。
	+ 为什么需要重新检查？如果该线程是第一个抢到锁的，那确实不需要。当某个线程出队调用signal，将该线程唤醒，进入同步队列，如果同步队列前面还有一个线程A会执行put操作，那么A将先获得锁，A被唤醒执行入队操作，然后当前线程竞争锁成功，而此时队列仍是满的。
+ 此`put`函数只有成功入队后，才可能从`put`调用处返回。
+ 当队列未满，则入队。

#### 超时offer

```java
public boolean offer(E e, long timeout, TimeUnit unit)
    throws InterruptedException {

    checkNotNull(e);
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length) {
            if (nanos <= 0)//如果队列是满的，且等待时间<= 0这代表不用等待，所以直接返回false
                return false;
            nanos = notFull.awaitNanos(nanos);
        }
        enqueue(e);
        return true;
    } finally {
        lock.unlock();
    }
}
```

从`notFull.awaitNanos(nanos)`返回有三种原因：超时前的signal、超时前的中断、超时。

- 超时前的signal。只有这种情况，才可能返回一个大于0的数字。
- 超时前的中断。返回时，抛出中断异常。
- 超时（不管之后有没有中断）。只可能返回一个小于0的数字。

` nanos = notFull.awaitNanos(nanos);`使得await的总时长不会大于传入参数`nanos`的原本值。

#### 总结

| 入队方法  | 是否等待               | 队列满时的处理                         | 返回值        | 返回值含义                                   |
| --------- | ---------------------- | -------------------------------------- | ------------- | -------------------------------------------- |
| add       | 一次尝试，从不等待     | 抛出异常                               | true          | 入队成功                                     |
| offer     | 一次入队尝试，从不等待 | 返回false                              | true<br>false | 入队成功<br>入队失败                         |
| put       | 入队尝试失败后，等待   | 不返回，进入同步队列等待<br>会响应中断 | void          | 只要正常返回就代表入队成功                   |
| 超时offer | 入队尝试失败后，等待   | 不返回，进入同步队列等待<br>超时返回   | true<br>false | 规定时间内，入队成功<br>规定时间内，未能入队 |

### 出队

#### peek

```java
public E peek() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return itemAt(takeIndex); // null when queue is empty
    } finally {
        lock.unlock();
    }
}
```

#### poll

```java
//不响应中断，一次尝试
public E poll() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return (count == 0) ? null : dequeue();
    } finally {
        lock.unlock();
    }
}

private E dequeue() {
    // assert lock.getHoldCount() == 1;当前已经获取了锁
    // assert items[takeIndex] != null;队列不为空
    final Object[] items = this.items;
    @SuppressWarnings("unchecked")
    E x = (E) items[takeIndex];
    items[takeIndex] = null;
    if (++takeIndex == items.length)//takeIndex右移，循环数组
        takeIndex = 0;
    count--;
    if (itrs != null)
        itrs.elementDequeued();
    notFull.signal();//通知阻塞在notFull条件队列上的第一个线程
    return x;
}
```

#### take

```java
//响应中断，尝试失败进入等待
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0)
            notEmpty.await();
        return dequeue();
    } finally {
        lock.unlock();
    }
}
```

#### 超时poll

```java
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0) {
            if (nanos <= 0)
                return null;
            nanos = notEmpty.awaitNanos(nanos);
        }
        return dequeue();
    } finally {
        lock.unlock();
    }
}
```

逻辑跟超时offer一样。

#### 总结

| 出队方法 | 是否等待               | 队列空时的处理                                | 返回值         | 返回值含义                                   |
| -------- | ---------------------- | --------------------------------------------- | -------------- | -------------------------------------------- |
| peek     | 一次获得尝试，从不等待 | 返回null                                      | 非null<br>null | 队列不空<br>队列空                           |
| poll     | 一次出队尝试，从不等待 | 返回null                                      | 非null<br>null | 队列不空<br>队列空                           |
| take     | 出队尝试失败后，会等待 | 不返回，进入条件队列等待                      | void           | 正常返回表示出队成功                         |
| 超时poll | 出队尝试失败后，会等待 | 未超时，则进入条件队列等待<br>超时，返回false | 非null<br>null | 规定时间内，出队成功<br>规定时间内，未能出队 |

### remove

该函数如果删除的不是队首元素，会涉及到整体移动的过程，会比较耗时。

```java
public boolean remove(Object o) {
    if (o == null) return false;
    final Object[] items = this.items;
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        if (count > 0) {
            final int putIndex = this.putIndex;
            int i = takeIndex;
            do {
                if (o.equals(items[i])) {
                    removeAt(i);
                    return true;
                }
                if (++i == items.length)//循环数组
                    i = 0;
            } while (i != putIndex);
        }
        return false;
    } finally {
        lock.unlock();
    }
}
```

```java
void removeAt(final int removeIndex) {
    // assert lock.getHoldCount() == 1;已经获取了锁
    // assert items[removeIndex] != null;要删除的位置不为null
    // assert removeIndex >= 0 && removeIndex < items.length;removeIndex在正常范围内
    final Object[] items = this.items;
    if (removeIndex == takeIndex) {//如果要删除的是队首
        items[takeIndex] = null;
        if (++takeIndex == items.length)
            takeIndex = 0;
        count--;
        if (itrs != null)
            itrs.elementDequeued();
    } else {
        // 如果要删除的是中间节点或者队尾，那需要将该节点后的所有节点向前移动一格
        final int putIndex = this.putIndex;
        for (int i = removeIndex;;) {
            int next = i + 1;
            if (next == items.length)
                next = 0;
            if (next != putIndex) {
                items[i] = items[next];
                i = next;
            } else {
                items[i] = null;
                this.putIndex = i;
                break;
            }
        }
        count--;
        if (itrs != null)
            itrs.removedAt(removeIndex);
    }
    notFull.signal();//通知notFull
}
```

### 总结

+ 当队列为空或为满时，takeIndex putIndex二者才会相同。
+ 所有常用操作都需要加锁，甚至是属于读操作的peek，因为加锁强制内存刷新，能让线程看到最新的队列。
+ 入队出队操作，都有一次尝试版本，和阻塞等待版本。
	使用Lock来控制并发操作。
+ 两个Condition的使用，是控制阻塞等待的关键。
+ 删除操作支持删除内部元素。