# DelayQueue

### 前言

`DelayQueue`是一个无界阻塞队列，它和`PriorityBlockingQueue`一样是一个优先队列，但区别在于队列元素只能放置Delayed对象，而且只有元素到期后才能将其出队。

内部是一个最小堆，堆顶永远是最先“到期”的那个元素。如果堆顶元素没有到期，即使线程发现队列中有元素，也不能将其出队。

`DelayQueue`需要依赖于元素对Delayed接口正确实现，即保证到期时间短的Delayed元素`.compareTo`(到期时间长的Delayed元素) < 0，这样可以让到期时间短的Delayed元素排在队列前面。

### 成员

```java
//非公平锁
private final transient ReentrantLock lock = new ReentrantLock();
//最小堆
private final PriorityQueue<E> q = new PriorityQueue<E>();
//Leader-Follower线程模式中的Leader，它总是等待获取队首
private Thread leader = null;
//不管哪种线程都将阻塞在这个条件队列上。但Follower可能是无限的阻塞
private final Condition available = lock.newCondition();
```

### Leader-Follower

首先我们想一个问题，在队列中的处于队首的Delayed元素，由于还没到期，只能暂时等待等到它到期，这种暂时等待必然需要使用到`Condition.awaitNanos`。虽然第一个来的线程是可以明确知道要等队首元素多久（通过`getDelay`），但第二个或以后来的线程就不知道该等多久了，明显它们应该去等待排名第二或以后的元素，但奈何优先队列是个最小堆，最小堆只能时刻知道最小元素是谁。

所以，干脆让第二个或以后来的线程无限阻塞（`Condition.await`），但我们让第一个线程负责唤醒沉睡在条件队列上的线程。因为第一个线程总是使用`Condition.awaitNanos`，所以不会造成条件队列上的线程睡到天荒地老。第一个线程总是等待获得队堆顶，当它出队成功后，再唤醒后面的线程去获得新堆顶。

上面说的第一个线程其实就是Leader-Follower模式中的Leader了，它总是会以`Condition.awaitNanos`的方式阻塞，这保证了它不会一直沉睡。而其他线程就是所谓的Follower，当它们检测到Leader的存在时，则可以放心使用`Condition.await`，就好像调好了闹钟所以可以放心大胆睡觉一样。

### 入队

#### offer

```java
public boolean offer(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        q.offer(e);
        if (q.peek() == e) {
            leader = null;
            available.signal();
        }
        return true;
    } finally {
        lock.unlock();
    }
}
```

- `lock.lock()`入队不响应中断，也没有必要响应中断。毕竟`DelayQueue`是无界队列，不可能出现因队列满而阻塞的情况，也就不用响应中断了。
- `if (q.peek() == e)`成立，说明新元素入队后成为了堆顶，说明最小元素更新了。那么之前的leader（如果存在的话）调用的`awaitNanos`的参数相较于新的最小元素偏大了，因为现在有了更小的元素进来。那么干脆清空leader（也有可能leader本来就是null，即使条件队列里有线程），唤醒条件队列第一个线程，让leader以更小的参数调用awaitNanos。
- `if (q.peek() == e)`不成立，说明之前的leader（如果存在的话）调用的`awaitNanos`的参数还是正确的，所以也就不需要什么操作。
- 理论上，该函数不可能失败，只会返回true。

### 出队

#### take

```java
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
            E first = q.peek();
            if (first == null)
                available.await();
            else {
                long delay = first.getDelay(NANOSECONDS);
                if (delay <= 0)
                    return q.poll();
                first = null; // 因为后面需要等待，阻塞期间不要持有引用，以免元素泄露
                if (leader != null)
                    //为什么从available.await返回后还需要检查leader是否为null？
                    //因为在poll方法可以不通过await来直接设置leader
                    //假设队列offer了第一个，available.signal，此时poll方法抢锁成功，当前线程进入同步队列
                    //poll方法设置leader之后还会调用available.await方法，只有等待结束后才会将leader置null
                    //poll调用await，释放锁，当前线程获得锁，从await返回，此时leader不为null
                    available.await();
                else {//设置当前线程为leader
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {//等待直到第一个到期
                        available.awaitNanos(delay);
                    } finally {//先将leader置为null，下一个循环就直接将第一个移出队列
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {//如果leader为null，那么available.signal
        if (leader == null && q.peek() != null)
            available.signal();
        lock.unlock();
    }
}
```

流程总结如下：

1. Leader执行`available.awaitNanos(delay)`，进行限时的阻塞。
2. Follower执行`available.await()`，进行无限的阻塞。
3. Leader线程在退出`take`函数时会唤醒一个沉睡在条件队列上的Follower，所以Follower实际上不会一直阻塞下去。
4. 每个线程在阻塞前都会尝试成为Leader，否则成为Follower。同时只有一个Leader。
5. Leader在阻塞期间一直都是Leader身份（Leader == 当前线程），但唤醒后马上清空掉自己的Leader身份（Leader = null），之后一段时间由于一直持有锁（这里指退出`take`或再次阻塞之前），所以也不用担心别的线程修改Leader。
	1. 如果 退出`take`，退出前将唤醒一个沉睡在条件队列上的Follower。
	2. 如果再次阻塞，那么重新获得Leader身份。反正一直持有着锁，当确定了要重新当Leader后再获得Leader身份也不迟。

#### 内存泄露

`take`函数中，`first = null`用来防止内存泄漏。简单的说，每个线程在阻塞期间都不持有堆顶元素的引用。

假设没有这句，看看内存泄漏是怎么发生的：

1. 线程A、B、C先后调用`take`。
2. 线程A是Leader，它唤醒后首先出队 堆顶元素。处理完这个元素后，元素原本应该被GC掉。
3. 线程BC还持有该元素引用。即使线程B马上被唤醒，线程C也还在阻塞中，必然这个元素不能被GC掉。
	造成了内存泄漏。

#### 超时poll

该函数最大的特点就是，无论哪种情况，阻塞都是使用`awaitNanos`进行限时的阻塞。

```java
//超时返回null
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
            E first = q.peek();
            if (first == null) {
                if (nanos <= 0)
                    return null;
                else
                    nanos = available.awaitNanos(nanos);
            } else {
                long delay = first.getDelay(NANOSECONDS);
                if (delay <= 0)
                    return q.poll();
                if (nanos <= 0)
                    return null;
                first = null; // 阻塞期间不保存引用
                if (nanos < delay || leader != null)
                    //nanos<delay，如果新offer的node的delay小于nanos，还是有机会的，否则无了
                    //leader!=null，需要等待
                    nanos = available.awaitNanos(nanos);
                else {
                    //执行到这里，有两种情况
                    //1、nanos>=delay，这种情况下肯定能等到的，那么强占leader，first由自己返回，符合非公平的语义
                    //2、leader==null，这种情况直接设置自己为leader
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        //执行到这，await返回后的下一个循环肯定能成功返回first
                        long timeLeft = available.awaitNanos(delay);
                        nanos -= delay - timeLeft;
                    } finally {
                        //将leader置null，还持有锁，不用担心被修改leader
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        if (leader == null && q.peek() != null)
            available.signal();
        lock.unlock();
    }
}
```

- 无论是线程为空时，或是当前线程是一个Follower时，都改用`available.awaitNanos(nanos)`进行限时的阻塞。

### 迭代器

和`PriorityBlockingQueue`一样，迭代器初始化时，传入一个当前`DelayQueue`队列的数组快照。所以也是弱一致性的。

### 总结

+ `DelayQueue`和`PriorityBlockingQueue`一样是一个优先队列。
+ 队列元素只能放置Delayed对象，而且只有元素到期后才能将其出队。
+ `DelayQueue`需要依赖于元素对`Delayed`接口正确实现。
+ Leader-Follower模式减小了无意义的线程唤醒，只在Leader退出出队函数时唤醒Follower，以避免Follower线程一直阻塞在AQS条件队列里。