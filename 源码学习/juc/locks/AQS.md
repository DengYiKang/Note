# AbstractQueuedSynchronizer

[TOC]

AQS继承自`AbstractOwnableSynchronizer`类，后者主要是维护了一个保存独占线程的变量以及对它的set/get方法。

默认的情况下，acquire操作一般发生在入队前，因此新的请求线程可能会比队列中的等待线程先一步获取锁，即非公平锁。如果想实现公平锁，那么可以重新设计`tryAcquire`或者`tryAcquireShared`方法，比如通过`hasQueuedPredecessors`方法的返回值来判断队列中是否有线程等待，如果有的话直接返回`false`。

在高并发的情况下，非公平锁的性能要比公平锁的要好，对于非公平锁，一个新来的线程如果成功获取了锁，那么可以直接执行其任务，而对于公平锁，需要进行若干操作（如对队列的操作，唤醒等调度）才能开始执行其任务，需要的时间比公平锁多。但是非公平锁容易导致线程饥饿的问题。

对AQS的基本用法是重写以下方法来满足需求：

+ `tryAcquire`或`tryAcquireShared`
+ `tryRelease`或`tryReleaseShared`
+ `isHeldExclusively`

例如，实现一个不可重入的互斥锁：

```java
class Mutex implements Lock, java.io.Serializable {

    //代理模式,代理Sync类
    private static class Sync extends AbstractQueuedSynchronizer {
        //判断是否有线程持有锁(state为1表示有线程持有锁)
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        //尝试获取锁
        public boolean tryAcquire(int acquires) {
            assert acquires == 1;
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        //尝试释放锁
        protected boolean tryRelease(int releases) {
            assert releases == 1;
            if (getState() == 0) throw new IllegalMonitorStateException();
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        //condition
        Condition newCondition() {
            return new ConditionObject();
        }

        //反序列化
        private void readObject(ObjectInputStream s)
                throws IOException, ClassNotFoundException {
            s.defaultReadObject();
            setState(0);
        }
    }

    //代理
    private final Sync sync = new Sync();

    public void lock() {
        sync.acquire(1);
    }

    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    public void unlock() {
        sync.release(1);
    }

    public Condition newCondition() {
        return sync.newCondition();
    }

    public boolean isLocked() {
        return sync.isHeldExclusively();
    }

    public boolean hasQueuedThreads() {
        return sync.hasQueuedThreads();
    }

    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }
}
```

下面是以个闭锁(`Latch`)的例子：

闭锁是共享锁，使用`tryAcquireShared`和`tryReleaseShared`方法。通过`signal`方法来使所有等待的线程继续执行。

```java
public class BooleanLatch {

    private static class Sync extends AbstractQueuedSynchronizer {
        boolean isSignalled() {
            return getState() != 0;
        }

        //尝试获取共享锁,因为是latch的实现，所以无需再对state进行修改（latch的状态是一次性的）
        protected int tryAcquireShared(int ignore) {
            return isSignalled() ? 1 : -1;
        }

        //释放锁，就是激活所有等待线程的操作
        protected boolean tryReleaseShared(int ignore) {
            setState(1);
            return true;
        }
    }

    private final Sync sync = new Sync();

    //是否被激活
    public boolean isSignalled() {
        return sync.isSignalled();
    }

    //激活所有等待的线程
    public void signal() {
        sync.releaseShared(1);
    }

    //等待
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
}
```

AQS的等待队列示意图：

```
      +------+  prev +-----+       +-----+
 head |      | <---- |     | <---- |     |  tail
      +------+       +-----+       +-----+
```

AQS所用的等待队列是CLH锁队列(Craig, Landin, and Hagersten)的一个变式。CLH锁队列通常用于自旋锁，在AQS中被用来实现阻塞同步。对于CLH队列，入队比较简单，只需更改尾结点。而出队比较复杂，不仅需要更新头结点，还需要考虑各个结点的后继结点（可能出现因为超时或中断等原因的取消）。

队列分为condition queue和sync queue。

当一个结点成功地acquire时，它就成为了首结点，因此首结点永远不会是取消状态的结点。

注意，`next`域在入队操作不会被初始化，只有它被用到时才会。因此不能通过`next`域是否为空来判断是否为`tail`结点（话说都有`tail`变量了为什么还要这样判断），因此如果某个结点的`next`域为空，那么我们可以查看`tail`的`prev`结点进行二次检查（`prev`结点在入队和出队时是同步更新的）。

```java
//将当前线程置空，将prev结点置空
//该节点前的所有结点将会被gc掉
private void setHead(Node node) {
    head = node;
    node.thread = null;
    node.prev = null;
}
```

### 独占锁的获取

```java
/**
 * 获取独占锁
 * tryAcquire尝试获取独占锁，如果失败，则acquireQueued进行自旋（再次尝试获取，失败则挂起）
 * 如果线程被唤醒，那么在这里返回false，线程继续执行
 * 如果线程被中断，那么在这里返回true，设置中断状态，并未对中断做相应的处理（只是告知用户有中断发生）
 * 注意，tryAcquire方法是子类需要重写的，否则抛出异常（在AQS类中放置子类没有重写而其本身的实现就是抛出异常）
 * 根据需求不同，如公平锁与非公平锁，tryAcquire方法的实现不同
 */
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        //如果被中断，则设置中断状态
        selfInterrupt();
}

/**
 * 将当前线程包装成一个node，加入到队列中
 * 返回当前线程对应的node
 */
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    //这一部分的逻辑与enq方法类似，只是做一个简单的尝试
    //如果尝试失败，那么入队的操作由完整的enq方法来完成
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}

/**
 * 入队的完整流程
 */
private Node enq(final Node node) {
    //自旋
    for (;;) {
        Node t = tail;
        //注意head为dummy node，采用懒初始化的方式
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}

/**
 * 独占锁的获取，非中断模式（即这个方法中不处理中断，而是通过返回值的方式告诉调用者在等待期间是否被中断）
 * 注意参数node必须已经入队，一般以acquireQueued(addWaiter(Node.EXCLUSIVE), arg)的方式被调用
 */
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            //head结点是假结点(dummy node)，其thread成员变量为null，被保存在exclusiveOwnerThread里
            //head结点对应的线程是持有锁的线程，当前驱是head时，即自身是等待队列中的第一个，允许请求锁
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                //请求成功
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            //shouldParkAfterFailedAcquire判断是否具有挂起的条件（即前驱状态是否为SIGNAL）
            //如果没有挂起条件，则会设置前驱状态为SIGNAL使其具有挂起条件(不断自旋，最终必定会返回true)
            //如果具有挂起的条件，则parkAndCheckInterrupt挂起，如果有中断，返回true，被唤醒，返回false
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())//在这里被阻塞
                //保存中断状态，待成功获得锁之后将是否被中断告诉调用者
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}

/**
 * 该方法一般在acquire失败后，挂起前调用，判断是否具备挂起的条件
 * 一个结点如果被挂起，只有它的前驱是SIGNAL状态时它才可能被唤醒
 * 因此如果前驱的状态是SIGNAL，则具备挂起条件，返回true
 * 如果前驱已经被取消(status>0)，那么跳过被取消的前驱，返回false
 * 如果前驱是其他状态(0或PROPAGATE)，那么把前驱设置为SIGNAL，返回false
 * 这个方法一般在自旋里调用，所以就算返回false，这个方法也会被重新调用直到前驱为SIGNAL（除非acquire成功）
 * 即如果需要acquire失败后且前驱的状态不为-1，第一次执行设置前驱为-1，第二次执行park自己
 * 为什么不一次性设置成SIGNAL并返回true呢？
 * 因为在某些情景下，前面的node可能都取消了，该node成为head后继，可以获取锁。
 * 相当于在阻塞前给最后一次自救的机会
 */
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)
        return true;
    if (ws > 0) {
        //跳过被取消的前驱
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        //设置前驱为SIGNAL
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}

/**
 * 挂起当前线程，直到被中断或者被唤醒(unpark)
 * 如果被中断，则返回true
 */
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

### 独占锁的释放

```java
/**
 * 独占锁的释放
 * 首先尝试释放锁tryRelease，tryRelease方法由子类实现
 * 如果tryRelease成功，检查head是否为空，或者状态是否为0，如果不为空且状态不为0（即SIGNAL），则唤醒后继结点
 * （首先head可能是取消状态的，其次独占锁没有PROPAGATE状态，因此这里其实是判断是否为SIGNAL状态）
 */
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            //唤醒后继结点
            unparkSuccessor(h);
        return true;
    }
    return false;
}

//唤醒node的后继结点,这个后继结点必须是非空或非取消状态的
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    //将该node的状态置为0
    //在release方法中调用时，该条件始终成立
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    /*
     * 将下一个结点unpark，如果下一个结点为null或者被取消了（next懒初始化）
     * 那么从tail结点往前找，找到离该节点最近的非null非取消状态的结点，将它unpark。
     */
    Node s = node.next;
    //s=null说明目前为中间状态,next还未设置好，可以根据prev指针来寻找（prev指针是绝对有效的）
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    //unpark后继节点对应的线程
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

### 独占锁和共享锁之间的方法对比

| 独占锁                                        | 共享锁                                              |
| --------------------------------------------- | --------------------------------------------------- |
| `tryAcquire(int arg)`                         | `tryAcquireShared(int arg)`                         |
| `tryAcquireNanos(int arg, long nanosTimeout)` | `tryAcquireSharedNanos(int arg, long nanosTimeout)` |
| `acquire(int arg)`                            | `acquireShared(int arg)`                            |
| `acquireQueued(final Node node, int arg)`     | `doAcquireShared(int arg)`                          |
| `acquireInterruptibly(int arg)`               | `acquireSharedInterruptibly(int arg)`               |
| `doAcquireInterruptibly(int arg)`             | `doAcquireSharedInterruptibly(int arg)`             |
| `doAcquireNanos(int arg, long nanosTimeout)`  | `doAcquireSharedNanos(int arg, long nanosTimeout)`  |
| `release(int arg)`                            | `releaseShared(int arg)`                            |
| `tryRelease(int arg)`                         | `tryReleaseShared(int arg)`                         |
| -                                             | `doReleaseShared()`                                 |

### 共享锁的获取

```java
/**
 * 共享锁的获取，不处理中断
 * tryAcquiredShared是子类需要重写的
 * 首先尝试获取共享锁tryAcquireShared(arg)，如果失败则调用doAcquireShared(arg)
 * 而doAcquireShared尝试获取锁直到成功为止
 */
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}

/**
 * 共享锁的获取，逻辑跟独占锁的acquireQueued方法类似
 */
private void doAcquireShared(int arg) {
    //将当前线程包装为共享模式的node
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        //自旋
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                //如果是等待队列中的第一个，那么允许尝试获取锁
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        //恢复中断状态
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            //这一块代码与独占锁的acquireQueued方法的完全一致
            //判断是否具有挂起的条件（即前驱状态是否为SIGNAL），否则设置前驱为SIGNAL
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())//在此阻塞
                //如果被中断过，则保存下来
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}

/**
 * 共享锁在获取时的处理跟独占锁不同
 * 共享锁在获取成功后会检查它的后继是否也能获取锁，如果能就unpark它
 * 参数propagate是tryAcquireShared(arg)的返回值，即当前剩余的锁资源
 * 如果propagete>0，那么尝试唤醒后继结点去请求锁
 * in shared mode, if so propagating if either propagate > 0 or
 * PROPAGATE status was set.
 */
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // Record old head for check below
    setHead(node);
    /*
     * Try to signal next queued node if:
     *   Propagation was indicated by caller,
     *     or was recorded (as h.waitStatus either before
     *     or after setHead) by a previous operation
     *     (note: this uses sign-check of waitStatus because
     *      PROPAGATE status may transition to SIGNAL.)
     * and
     *   The next node is waiting in shared mode,
     *     or we don't know, because it appears null
     *
     * The conservatism in both of these checks may cause
     * unnecessary wake-ups, but only when there are multiple
     * racing acquires/releases, so most need signals now or soon
     * anyway.
     */
    //h肯定不为null，在调用该方法前已经执行过了addWaiter，队列至少有一个node
    //第一个h.waitStatus判断，老结点h可能不为0的，有可能在setHead方法之前有个线程释放锁了，
    //通过doReleaseShared方法对status=0的老结点更改status为PROPAGATE状态
    //话说第二个h.waitStatus<0的判断条件是在head不是队尾时肯定成立吧？那PROPAGETE的意义在哪
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            //唤醒head的后继
            doReleaseShared();
    }
}

private void setHead(Node node) {
    head = node;
    //注意，head结点的thread变量为空，因为线程acquire成功或者被阻塞过但被唤醒且acquire成功，没必要存储了
    //事实上，head就是一个dummy node，head的后继才是等待队列中的第一个node
    node.thread = null;
    node.prev = null;
}

/*
 * 如果head的状态为SIGNAL，则把状态置为0，然后唤醒后继节点(unparkSuccessor(h))
 * 如果head的状态为0，则把状态置为PROPAGATE
 */
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;
        }
        if (h == head)
            break;
    }
}
```

### 共享锁的释放

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        //doReleaseShared在上面提到过，如果head的状态为SIGNAL，那么置为0，唤醒后继
        //如果head的状态为0，那么置为PROPAGETA
        doReleaseShared();
        return true;
    }
    return false;
}
```

### PROPAGATE

> 先执行LockSupport.unpark(), 再执行 LockSupport.park(), 这个时候线程是park不住的。比如当D进入到第二次的`shouldParkAfterFailedAcquire`方法中时，前驱C的status肯定为-1，此时C进行release，并对D进行唤醒，那么有两种情况：1、D先进行unpark，C后进行唤醒D。2、C先唤醒D（然而D现在并没有被阻塞），D后进行unpark。如果没有这种性质的保证，第二种情况下D将永远无法被唤醒。

考虑没有PROPAGATE状态，即`doReleaseShared()`方法中删除以下语句：

`else if (ws == 0 && !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))`

先假设有4个线程：A，B，C，D。A，B持有锁，并在某时释放锁，C，D请求锁，此时C为head，D为队列中的第一个，A已经释放了锁，B仍持有锁。

~~需要考虑各种时间点的组合，这些时间点为：A，B，C的`doReleaseShared()`，C的`setHeadAndPropagate`方法的`setHead`时间点与`if`判断时间点，D的两次`shouldParkAfterFailedAcquire`时间点等等。~~

在公平竞争的条件下，队列不会hang住。假设A，B是依次进行release的，C先于D acquire。D在调用第二次`shouldPark...`方法之前有一个自救机会。假设D会被阻塞，那么意味着在阻塞之前B、C都还未release。由于第一次`shouldPark...`方法已经将C的status设置为SIGNAL了，因此C或B release的时候必然会唤醒D。

~~但上述情况是在公平竞争的条件下。即公平竞争时不会发生队列被hang住的情况。~~

~~考虑在非公平竞争的条件下，当B的release和C的release都发生在D的第一次`shouldParkAfterFailedAcquire`的同时但设置C为SIGNAL之前，即B 和C release的时候，D在队列里，但是还没有将C的status设置为SIGNAL(**即C的status仍为0**)，那么B和C的release将不会unpark D，D只能通过自己acquire成功才能避免阻塞，一旦阻塞就没有其他线程来唤醒自己了（因为前驱已经release了）。在第二次`shouldParkAfterFailedAcquire`方法被调用之前有一次自救，即tryAcquire，在公平竞争的条件下，因为D是队列中的第一个，所以必定acquire成功（有个常见的误区，就是在队列中的就是被阻塞的，这是错误的。先执行addwaiter方法，这个时候就加入队列了，后面还有两次自旋，第二次`shouldPark..`方法才会将自己阻塞）；但是在非公平竞争的条件下，如果这时有2个新的线程来请求锁，因为非公平竞争，它们在入队之前会先tryAcquire，当它们获取成功时，D便获取失败，那么就只能被阻塞了，后续永远不会被唤醒，队列hang住。~~我错了，其实不会产生bug。这两个新线程并不会加入队列，head还是C，而C的status是SIGNAL，而那两个新线程release的时候会通过判断head即C的状态是否为SIGNAL来唤醒D，只要那两个新线程release了，D就能被唤醒。

所以引入`PROPAGATE`并不是为了解决bug状态？可能是加速？

加速的重点在于`setHeadAndPropagate`方法（好像只有这个方法关联到了`PROPAGATE`）。`setHeadAndPropagate`的`if`判断条件其中之一为，如果旧head的status小于0，那么就唤醒新head的后继（前面有propagate>0短路，用到这个判断的时候意味着propagete=0）。注意，能调用`setHeadAndPropagate`那么意味着新head没有被阻塞过或是阻塞过后被唤醒，那么意味着如果没有其它线程release的话，**旧head的status等于0**，那么自然不需要唤醒后继了。但是考虑这一种情况，**在`setHeadAndPropagate`的调用时间点与其里面的`setHead`方法调用时间点之间**，其它线程release了，那么实际上state>0，而临时变量propagate=0，那么这种情况下，为了提高效率是应该唤醒新head的后继的。那么如何判断期间是否有别的线程release了呢？当然是根据旧head的status，期间若没有别的线程release，那么旧head的status=0，如果有，那么旧head的status为PROPAGATE<0，因此判断条件成立，唤醒后继。

### Condition接口的实现

#### 与Object自带的wait/notify对比

将Object自带的wait/notify方法与Condition提供的await/signal进行对比：

+ wait方法必须处于同步代码块中，换言之，调用wait方法必须已经获得了监视器锁；调用await的线程必须已经获得了lock锁。
+ 执行wait时，当前线程会释放已经获得的监视器锁，进入到该监视器锁的等待队列；执行await时，当前线程会释放已经获得的lock锁，进入该Condition的条件队列中。
+ 退出了wait后，当前线程又获得了监视器锁；退出await后，当前线程又获得了lock锁。
+ 调用监视器的nofity，会唤醒等待该监视器的线程，这个线程会重新竞争，竞争成功后会从wait方法处恢复执行；调用Condition的signal，会将条件队列的首位线程或所有线程加入到同步队列（或是在加入同步队列之后设置前驱SIGNAL失败时直接唤醒去竞争），这些线程此后重新开始竞争，竞争成功后，会从await方法处恢复执行。

#### 同步队列和条件队列

**同步队列**：

同步队列是一个双向链表，`nextWaiter`成员最多在共享模式下用来标识是否为共享锁结点。head是一个dummy node，其thread成员为null，第一个等待线程是head的后继。

**条件队列**：

```java
public class ConditionObject implements Condition, java.io.Serializable {
    private static final long serialVersionUID = 1173984872572414699L;
    //在获得锁的前提下调用await/signal，因此不需要volatile+CAS
    private transient Node firstWaiter;
    private transient Node lastWaiter;
}
```

条件队列是一个单向链表，使用`nextwaiter`作为链接（在同步队列中，`nextwaiter`最多在共享模式下用来标识是否为共享锁结点）。不存在dummy node。如果结点的状态为`CONDITION`，说明线程还在等待在这个Condition对象上；如果不是`CONDITION`的，说明这个结点已经前往同步队列。

#### 两者的关系

一个同步队列可以对应多个条件队列，signal和signalAll方法会将条件队列中的结点移到同步队列的末位去。

+ 一个结点刚入队同步队列，说明这个线程没有获得锁（尝试获取失败）。
+ 一个结点刚出队同步队列（指代表线程不在同步队列中的任何结点上，因为它被保存到`exclusiveOwnerThread`变量里了），说明这个线程刚获得了锁。
+ 如果一个结点刚入队条件队列，说明线程此时有锁，但即将释放（await）。
+ 如果一个结点刚出队条件队列，此时线程前往同步队列，此时没有获得锁。
+ 同步队列的结点的初始状态为0，条件队列的结点的初始状态为CONDITION

#### await

```java
//中断模式
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    //将当前线程包装成CONDITION状态的node加入到条件队列
    Node node = addConditionWaiter();
    //先释放所有的锁，并且保存到savedState变量，在被唤醒后重新请求相同数量的锁
    int savedState = fullyRelease(node);
    //中断级别
    int interruptMode = 0;
    //循环条件是判断是否在同步队列上
    //如果在同步队列上，意味着因为这个Condition被调用了signal而加入到同步队列
    //第一次循环必定被阻塞
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        //被唤醒有两种情况：
        //1、通过signal进入到同步队列中，后当自身为head的后继被唤醒
        //2、被别的线程中断
        //checkInterruptWhileWaiting方法将node设置状态为0，入队同步队列，返回中断等级
        //如果被中断过（即！=0），则break
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    //独占锁的获取（可以看出条件队列的node使用的是独占锁）
    //acquireQueued返回的bool值表示是否被中断过，savedState表示之前释放的锁要重新获取回来
    //interruptMode为0时，表示没有被中断（即由signal加入到同步队列的）。如果acquireQueued中断过
    //那么将interruptMode升级为REINTERRUPT
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        //interruptMode为THROW_IE时，抛出异常
        //interruptMode为REINTERRUPT时，重新设置中断信息
        reportInterruptAfterWait(interruptMode);
}
```

```java
//将当前线程包装成CONDITION状态的结点加入到条件队列
private Node addConditionWaiter() {
    Node t = lastWaiter;
    //如果末位结点被取消了（即不是CONDITION状态），则清理
    if (t != null && t.waitStatus != Node.CONDITION) {
        unlinkCancelledWaiters();
        t = lastWaiter;
    }
    //包装成CONDITION状态的node
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    lastWaiter = node;
    return node;
}
```

```java
//将当前线程持有的所有锁释放，返回释放的数量
final int fullyRelease(Node node) {
    boolean failed = true;
    try {
        int savedState = getState();
        if (release(savedState)) {
            failed = false;
            return savedState;
        } else {
            throw new IllegalMonitorStateException();
        }
    } finally {
        if (failed)
            node.waitStatus = Node.CANCELLED;
    }
}
```

```java
//检查是否被中断过，返回中断等级
//如果被中断过，那么将node状态设为0，入队同步队列
//THROW_IE表示中断流程发生在signal之前
//（中断流程是指将CONDITION置为0，入队同步队列，等待的操作，而不是指中断）
//正常的流程是signal使node进入同步队列，而不是中断流程。因此如果为THROW_IE，后续应该抛出异常
//REINTERRUPT表示中断流程发生在signal之后
private int checkInterruptWhileWaiting(Node node) {
    return Thread.interrupted() ?
        (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :
        0;
}
```

```java
//取消等待后，将node状态设为0，将它入队同步队列
//如果另一个线程先调用了signal方法（即上述所说的中断流程发生在signa流程之后），返回false，否则true
final boolean transferAfterCancelledWait(Node node) {
    if (compareAndSetWaitStatus(node, Node.CONDITION, 0)) {
        enq(node);
        return true;
    }
    //如果上面的CAS失败，说明有另一个线程调用了signal，因此需要等待它使node完全入队
    //否则后续的acquireQueued方法可能在node还未完全入队的情况下执行
    while (!isOnSyncQueue(node))
        Thread.yield();
    return false;
}
```

#### signal

```java
//对条件队列中的所有node进行signal操作
//signal操作会将对应的node的状态设为0，并且入队同步队列，设置前驱为SIGNAL
//如果无法保证前驱为SIGNAL，那么直接将node唤醒，由之后的acquireQueued来设置
public final void signalAll() {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignalAll(first);
}
```

```java
private void doSignalAll(Node first) {
    lastWaiter = firstWaiter = null;
    do {
        Node next = first.nextWaiter;
        //方便gc
        first.nextWaiter = null;
        //将first node的状态设为0，并入队同步队列
        transferForSignal(first);
        first = next;
    } while (first != null);
}
```

```java
//signal单个node
public final void signal() {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}
```

```java
final boolean transferForSignal(Node node) {
    //CAS失败表示中断流程先行发生
    //哪个流程成功了，哪个流程入队
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;

    //入队同步队列
    Node p = enq(node);
    int ws = p.waitStatus;
    //如果前驱被取消或者设置前驱SIGNAL失败，那么将它直接让唤醒，进行后续的acquireQueued方法
    //后续的acquireQueued能保证前驱为SIGNAL状态
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);
    return true;
}
```

#### await总结

从实现内部的持有锁情况来看：

- `await()`在从开头到`fullyRelease`执行前，是持有锁的。
- `await()`在从`fullyRelease`执行后 到 `acquireQueued`执行前，是没有持有锁的。
- `await()`在 `acquireQueued`执行后到最后，是持有锁的。
- `signal() signalAll()`全程都是持有锁的。

`await()`的整体流程如下：

1. 将当前线程包装成一个node后（`Node node = addConditionWaiter()`），完全释放锁（`int savedState = fullyRelease(node)`）。
2. 当前线程阻塞在`LockSupport.park(this)`处，等待signal线程或者中断线程的到来。
3. 被唤醒后，到达`acquireQueued`之前，当前线程的node已经置于`sync queue`之上了。
4. 执行`acquireQueued`，进行阻塞式的抢锁。
5. 退出`acquireQueued`时，当前线程已经重新获得了锁，之后进行善后工作。

#### 其他版本的await

```java
//非中断模式
//如果有中断发生，检测到后重新设置中断信息
//退出while循环（也可以理解为退出await）只能是别的线程进行signal
public final void awaitUninterruptibly() {
    Node node = addConditionWaiter();
    int savedState = fullyRelease(node);
    boolean interrupted = false;
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        if (Thread.interrupted())
            interrupted = true;
    }
    if (acquireQueued(node, savedState) || interrupted)
        selfInterrupt();
}
```

```java
//超时机制，如果超时，那么直接执行acquireQueued
//返回值为nanosTimeout-花费在此方法上的时长
public final long awaitNanos(long nanosTimeout)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter();
    int savedState = fullyRelease(node);
    final long deadline = System.nanoTime() + nanosTimeout;
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        //超时直接设置状态为0，入队同步队列
        if (nanosTimeout <= 0L) {
            transferAfterCancelledWait(node);
            break;
        }
        //如果剩余时间短，直接自旋就好了，没必要阻塞
        if (nanosTimeout >= spinForTimeoutThreshold)
            LockSupport.parkNanos(this, nanosTimeout);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
        //
        nanosTimeout = deadline - System.nanoTime();
    }
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null)
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
    return deadline - System.nanoTime();
}
```

```java
//超时返回true；未超时返回false
//注意，与awaitNanos的返回值区别开，两者不能等价
//此方法的计算范围只在while循环内，而awaitNanos计算范围是整个方法
//可能出现此方法返回true而awaitNanos返回负值的情况
public final boolean await(long time, TimeUnit unit)
        throws InterruptedException {
    long nanosTimeout = unit.toNanos(time);
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter();
    int savedState = fullyRelease(node);
    final long deadline = System.nanoTime() + nanosTimeout;
    boolean timedout = false;
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        if (nanosTimeout <= 0L) {
            timedout = transferAfterCancelledWait(node);
            break;
        }
        if (nanosTimeout >= spinForTimeoutThreshold)
            LockSupport.parkNanos(this, nanosTimeout);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
        nanosTimeout = deadline - System.nanoTime();
    }
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null)
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
    return !timedout;
}
```

```java
//注意这个没有自旋优化，所以调用这个方法时，最好给定的绝对时间比较远
//超时返回true，否则返回false
public final boolean awaitUntil(Date deadline)
        throws InterruptedException {
    long abstime = deadline.getTime();
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter();
    int savedState = fullyRelease(node);
    boolean timedout = false;
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        if (System.currentTimeMillis() > abstime) {
            timedout = transferAfterCancelledWait(node);
            break;
        }
        LockSupport.parkUntil(this, abstime);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null)
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
    return !timedout;
}
```