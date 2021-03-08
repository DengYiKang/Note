#### FutureTask

### 前言

`FutureTask`的使用方法已经在上一篇进行了讲解，其实它和`SynchronousQueue`很像，执行task的线程是生产者，获取执行结果的线程是消费者，消费者阻塞的原因不是生产者还没来，是因为生产者还没有生产出来执行结果。只不过，这里只有一个生产者（`FutureTask`对象），却可以对应到多个消费者（对同一个`FutureTask`对象调用get的不同线程）。

为了方便称呼，本文以生产者和消费者来指代两种线程。

### 状态

`FutureTask`的重点在于对task（`Callable.call()`）的执行的管理，而`FutureTask`通过一个volatile的int值来管理task的执行状态。

```java
private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;
```

     * Possible state transitions:
     * NEW -> COMPLETING -> NORMAL
     * NEW -> COMPLETING -> EXCEPTIONAL
     * NEW -> CANCELLED
     * NEW -> INTERRUPTING -> INTERRUPTED
可能的状态转移过程是上面四种。

![在这里插入图片描述](../../../pic/39)

初始化时`FutureTask`的state是NEW，这是构造器保证的。

+ COMPLEING状态表示正在设置outCome变量，有两种情况：
	+ task已经执行完毕，返回了结果，现在正在将结果保存在outCome变量
	+ task遇到了异常，现在正在将异常保存在outCome变量

这种状态的转换都是不可逆的，某些过程中还可能存在中间状态，但这种中间状态存在的时间很短，且马上也会变成相应的最终状态。所以可以认为，只要状态不是NEW的话，就可以认为生产者执行task已经完毕。关于这一点，`isDone`函数说明了这一点：

```java
 public boolean isDone() {
	return state != NEW;
}
```

调用这些函数可以使得状态发生转移（具体过程后面讲解）：

- `set(V v)`使得NEW -> COMPLETING -> NORMAL。
- `setException(Throwable t)`使得NEW -> COMPLETING -> EXCEPTIONAL。
- `cancel(boolean mayInterruptIfRunning)`可能有两种状态转移：
	- 当`mayInterruptIfRunning`为false时，使得NEW -> CANCELLED。
	- 当`mayInterruptIfRunning`为true时，使得NEW -> INTERRUPTING -> INTERRUPTED。

### 消费者链表

前面提到，对同一个`FutureTask`对象调用get的不同线程的都属于消费者，当生产者还没有执行完毕task时，调用get会阻塞。而做法是将消费者线程包装成一个链表节点，放到一个链表中，等到task执行完毕，再唤醒链表中的每个节点的线程。这种做法类似于AQS的条件队列和signalAll。

反正最终链表上的所有节点都将被唤醒，所以链表是栈的逻辑结构，这样只用保存栈顶head指针，稍微简单一点。

```java
static final class WaitNode {
    volatile Thread thread;
    volatile WaitNode next;
    WaitNode() { thread = Thread.currentThread(); }
}
```

### 成员

```java
/** task执行状态 */
private volatile int state;
 /** task */
private Callable<V> callable;
/** 执行结果，可能是泛型类型V 或 抛出的异常。这都是前两种状态转移才会设置的 */
private Object outcome;// non-volatile的，因为最终会对state进行CAS操作，从而保证可见性
/** 执行task的生产者 */
private volatile Thread runner;
/** 消费者栈的head指针 */
private volatile WaitNode waiters;
```

- `outcome`是Object类型，可以存任何类型对象。这样既可以存泛型类型V，也可以存异常对象。
- 当调用`new Thread(FutureTask对象).start()`时，生产者线程便创建并开始运行了，并且会在`FutureTask#run()`的刚开始就把生产者线程存放到`runner`中。
- 当调用`FutureTask对象.get()`时，如果task还未执行完毕，当前消费者线程会被包装成一个节点扔到栈中去。

### 构造器

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}

public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}
```

两个构造器都保证了初始时状态为NEW。除了可以接受`Callable`之外，还可以接受`Runnable`，但也是马上通过适配器模式把`Runnable`包装成一个`Callable`而已。

### 实现Runnable接口

![](../../../pic/38)

先来看一看`FutureTask`对`Runnable`接口的实现。

```java
public void run() {
    // 执行run函数的前提是state为NEW，且生产者位置还没有被占用
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {//再次检查
            V result;
            boolean ran;
            try {
                result = c.call();//执行task
                ran = true;
            } catch (Throwable ex) {//执行过程中抛出了异常
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)//顺利执行完毕
                set(result);
        }
    } finally {
        // 执行完毕释放生产者线程对象引用
        runner = null;
        // 消费者线程可能使得task取消，其中一种状态转移是NEW -> INTERRUPTING -> INTERRUPTED
        // 这种转移有中间状态，需要检测这种中间状态变成最终状态
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```

在函数的一开始，需要检测`state`是否为NEW，且当前线程对象需要占领`runner`，并且在退出run函数之前，一直都会占领着`runner`。

生产者自己执行task时（`c.call()`），有两种情况：

- 顺利执行完task，然后调用`set(result)`。
- 执行task途中抛出异常，然后调用`setException(ex)`。

#### set

```java
//将结果保存到outCome成员变量中
//这里的COMPLETING状态表示task已经执行完了，正在保存结果到outCome变量中
protected void set(V v) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        //需要检查是否为NEW
        //可能消费者线程调用了cancel方法将状态设置为INTERRUPTING、INTERRUPTED或CANCELED，这时不能设置结果
        outcome = v;
        UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
        finishCompletion();
    }
}
```

#### setException

```java
//将异常保存到outCome成员变量中
//这里的COMPLETING状态表示task遇到了异常，正在保存异常到outCome变量中
protected void setException(Throwable t) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        outcome = t;
        UNSAFE.putOrderedInt(this, stateOffset, EXCEPTIONAL); // final state
        finishCompletion();
    }
}
```

#### finishCompletion

```java
//唤醒等待队列的所有线程
private void finishCompletion() {
    for (WaitNode q; (q = waiters) != null;) {//先保存副本
        if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {//再将等待队列清空
            for (;;) {
                Thread t = q.thread;
                if (t != null) {
                    q.thread = null;
                    LockSupport.unpark(t);
                }
                WaitNode next = q.next;
                if (next == null)
                    break;
                q.next = null; // unlink to help gc
                q = next;
            }
            break;
        }
    }

    //空实现，用于给使用者扩展
    done();

    callable = null;        // to reduce footprint
}
```

此函数负责唤醒所有消费者线程，原理很简单，内层循环遍历链表的每个节点，唤醒每个节点的线程对象。

最后还会调用`done`函数，但这只是空实现，这是用来给使用者拓展用的，可以让生产者线程在执行完毕前多做一点善后工作。

#### handlePossibleCancellationInterrupt

```java
finally {
        // 执行完毕释放生产者线程对象引用
        runner = null;
        // 消费者线程可能使得task取消，其中一种状态转移是NEW -> INTERRUPTING -> INTERRUPTED
        // 这种转移有中间状态，需要检测这种中间状态变成最终状态
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
```

最后还有finally块，用于处理task被消费者取消的状态。

```java
//自旋等待直到自身的状态从INTERRUPTING变成INTERRUPTED状态\
//注意，这个最终状态的设置是由消费者完成的
//即该线程需要等待消费者设置完毕才能做后续的处理
private void handlePossibleCancellationInterrupt(int s) {
    if (s == INTERRUPTING)
        while (state == INTERRUPTING)
            Thread.yield();
}
```

如果是中间状态，那么调用`handlePossibleCancellationInterrupt`自旋等待直到最终状态。

### 实现Future接口

#### 普通get、超时get

```java
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
    return report(s);
}
```

此函数判断当前`state`，并根据情况调用`awaitDone`进行阻塞等待。

+ 如果`state`为NEW，那么生产者还没开始执行呢，肯定得阻塞等待。
+ 如果`state`为COMPLETING，那么生产者线程马上执行完了，并且是 生产者正常执行完的过程（即前两种状态转移），那么也阻塞等待，即使马上会被唤醒。

```java
//等待超时将抛出异常
public V get(long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException {
    if (unit == null)
        throw new NullPointerException();
    int s = state;
    if (s <= COMPLETING &&//如果还没完成
        (s = awaitDone(true, unit.toNanos(timeout))) <= COMPLETING)//等待一段时间，还没完成就抛出异常
        throw new TimeoutException();
    return report(s);
}
```

#### awaitDone

```java
//等待，直到被signal
private int awaitDone(boolean timed, long nanos)
    throws InterruptedException {
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    WaitNode q = null;
    boolean queued = false;
    for (;;) {
        if (Thread.interrupted()) {//如果线程被中断，则将对应的等待节点删除，抛出异常
            removeWaiter(q);
            throw new InterruptedException();
        }

        int s = state;
        if (s > COMPLETING) {//如果执行完了，那么将等待节点的线程置为null
            if (q != null)
                q.thread = null;
            return s;
        }
        else if (s == COMPLETING) //快要完成了，那就先让出cpu，下个循环可能就完成了
            Thread.yield();
        else if (q == null)//如果没有完成，那么先新建等待节点，下个循环还没完成，那就该节点入队了
            q = new WaitNode();
        else if (!queued)
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                 q.next = waiters, q);//将等待节点入队
        else if (timed) {//执行到这里，等待节点已经入队。如果有超时机制的话，等待对应时间
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                removeWaiter(q);
                return state;
            }
            LockSupport.parkNanos(this, nanos);
        }
        else//无超时机制，那么无限阻塞
            LockSupport.park(this);
    }
}
```

#### removeWaiter

```java
//注意，先将本节点的thread置为null，再进行清理
//因为并发，同一时间，等待队列中可能存在多个thread为null的节点，因此需要从头到尾清理
private void removeWaiter(WaitNode node) {
    if (node != null) {
        node.thread = null;
        retry:
        for (;;) {          // restart on removeWaiter race
            for (WaitNode pred = null, q = waiters, s; q != null; q = s) {//往后移动q，pred->q->s
                s = q.next;
                if (q.thread != null)
                    pred = q;
                else if (pred != null) {//执行到这里，q.thread=null，将q unlink，变成pred->s
                    pred.next = s;
                    if (pred.thread == null) //如果pred.thread也是null，那么需要将pred unlink，跳到外循环，重新读取waiters
                        continue retry;
                }
                else if (!UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                      q, s))
                    //执行到这里，q.thread=null，pred.thread=null，那么设置s为新waiter，跳到外循环，重新读取waiters
                    continue retry;
            }
            break;
        }
    }
}
```

这里设置一个节点的`next`变量不需要CAS，如果多个线程同时修改`next`或访问`next`：

+ 同时修改的情况：`next`域肯定不会跨过未删除的节点，可能会指向已经被删除的节点，即可能为`pred->node.thread=null->...->node.thread!=null->...`或者`pred->node.thread!=null->...`，这些情况都是允许、合法的。
+ 修改的同时读的情况：某一节点被删除，但是另一个线程恰好读取到了这个节点。就算当前节点已经被unlink了，但只是从队列上出发找不到被unlink的节点，从unlink的节点出发还是能找到队列的，因此无关紧要。

#### report

在get方法里，当`awaitDone`返回后，`return report(s);`，此时`s`已经是最终状态。

当task正常执行完后，返回结果，当被取消或者遇到了异常，那么直接抛出对应的异常。

```java
private V report(int s) throws ExecutionException {
    Object x = outcome;
    if (s == NORMAL)
        return (V)x;
    if (s >= CANCELLED)
        throw new CancellationException();
    throw new ExecutionException((Throwable)x);
}
```

#### cancel

```java
//mayInterruptIfRunning表示是否需要用中断的方式
public boolean cancel(boolean mayInterruptIfRunning) {
    //返回false有以下几种情况
    //1、state!=NEW：这种情况要么是正在设置结果的中间状态，要么是最终状态
    //2、state=NEW，但CAS state失败，说明状态变化了或者其他线程调用了cancel
    
    //如果需要中断，就设置为INTERRUPTING（中间状态），否则设置为CANCELLED（最终状态）
    if (!(state == NEW &&
          UNSAFE.compareAndSwapInt(this, stateOffset, NEW,
              mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
        return false;
    //到这一步，CAS设置state成功
    try {
        if (mayInterruptIfRunning) {//如果需要中断，那就中断，中断完设置state为最终状态
            try {
                Thread t = runner;
                if (t != null)
                    t.interrupt();
            } finally { // final state
                UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
            }
        }
    } finally {
        //唤醒等待的所有线程
        finishCompletion();
    }
    return true;
}
```

消费者实际上不可能使得正在执行的生产者线程终止掉。当`mayInterruptIfRunning`为false时，`cancel`方法只是将`state`设置为CANCELLED便返回；当`mayInterruptIfRunning`为true时，`cancel`中断了正在执行的线程。但是，中断一个正在运行的线程，线程的运行状态不会发生变化，只是会设置以下线程的中断状态。也就是说，这也不能使生产者线程运行终止。除非生产者线程运行的代码`Callable.call`时刻在检测自己的中断状态并做相应处理。

`cancel`既然不能终止生产者线程，那么有什么用？

+ 如果参数为true，那么会去中断生产者线程。但生产者线程能否检测到，取决于生产者线程运行的代码（`Callable.call()`）。
+ 状态肯定会变成CANCELLED或INTERRUPTED，新来的**消费者**线程会直接发现，然后在`get`函数中不去调用`awaitDone`。
+ 对于**生产者**线程来说，执行task期间不会影响。但最后执行`set`或`setException`，会发现这个state，然后不去设置`outcome`。

#### isCancelled

```java
public boolean isCancelled() {
    return state >= CANCELLED;
}
```

#### isDone

注意，`isDone`包含各种中间状态和最终状态，无论是否顺利执行完毕。

```java
public boolean isDone() {
    return state != NEW;
}
```

### 普通写和CAS写混合

在实现中可以看到有时候使用普通写的语义，有时候使用CAS写。

```java
public boolean cancel(boolean mayInterruptIfRunning) {
    if (!(state == NEW &&
          UNSAFE.compareAndSwapInt(this, stateOffset, NEW,
              mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
        return false;
    try {    // in case call to interrupt throws exception
        if (mayInterruptIfRunning) {
            try {
                Thread t = runner;
                if (t != null)
                    t.interrupt();
            } finally { // final state
                UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
            }
        }
    } finally {
        finishCompletion();
    }
    return true;
}
```

比如`cancel`函数中的`UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED)`这里使用的普通写，其他线程可能不能马上看到，但这没关系。因为：

- 一来，这个状态转移是唯一的。INTERRUPTING只能变成INTERRUPTED。其他线程暂时看不到 INTERRUPTED 也没关系，因为只有`cancel`方法需要用CAS判断state是否为INTERRUPTING，其他线程的CAS必然无法成功。
- 二来，`finishCompletion`中也有对其他volatile字段的CAS写操作。这样做会把之前的普通写都刷新到内存中去。（？）

### 总结

- `FutureTask`整合了`Callable`对象，使得我们能够异步地获取task执行结果。
- 执行`FutureTask.run()`的线程就相当于生产者，生产出执行结果给`outcome`。执行`FutureTask.get()`的线程就相当于消费者，它们会阻塞等待直到执行结果产生。
- 如果生产者线程已经开始执行`Callable.call()`，那么消费者调用`cancel`，实际上是无法终止生产者的运行的。

