# ThreadPoolExecutor

### 前言

`ThreadPoolExecutor`提供了管理线程的功能，它的最大好处在于能够复用线程，而不必在想要异步执行时用原始的`new Thread().start()`起线程。在构造`ThreadPoolExecutor`时通过给定不同的参数，可以得到不同效果的线程池。这样，使用者可以不用关心线程的管理，而专心于业务的编写。

### 成员和构造器

#### 线程池状态

`ctl`变量把int型的bit分为了两部分：

- 高3位bit作为线程池的状态。
- 低29位bit作为线程数量。

而这样做的好处就是，把对两种状态的改变 变成了对同一个变量的CAS操作，毕竟单个CAS操作才是原子的。

```java
private static int ctlOf(int rs, int wc) { return rs | wc; }//将线程池状态 和 线程数量 合并起来

//初始时，线程池状态为RUNNING；线程数量为0
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;// 32-3 = 29
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;// 000111...111(29个1)

// 线程池状态在高3bit中
private static final int RUNNING    = -1 << COUNT_BITS;// 0b111(忽略后面的29个0)
private static final int SHUTDOWN   =  0 << COUNT_BITS; // 0b000
private static final int STOP       =  1 << COUNT_BITS;// 0b001
private static final int TIDYING    =  2 << COUNT_BITS;// 0b010
private static final int TERMINATED =  3 << COUNT_BITS;// 0b011

//阻塞队列
private final BlockingQueue<Runnable> workQueue;
//非公平锁，用于限制对worker set的并发访问
private final ReentrantLock mainLock = new ReentrantLock();
//只能在获取锁的前提下访问worker set
private final HashSet<Worker> workers = new HashSet<Worker>();
//用于终止await
private final Condition termination = mainLock.newCondition();
```

不难发现，这几个线程池状态无论后面的29位bit是什么，它们的大小顺序是完全的从小到大的顺序。

![在这里插入图片描述](../../../pic/41)

上图体现线程池状态转换的关系，注意，这些转换都是不可逆的。

### 构造器

```java
/**
 * @param corePoolSize    核心线程数
 * @param maximumPoolSize 最大线程数
 * @param keepAliveTime   空闲线程等待超时时间
 * @param unit            超时时间单位
 * @param workQueue       阻塞任务队列
 * @param threadFactory   线程工厂
 * @param handler         拒绝策略
*/
public ThreadPoolExecutor(int corePoolSize,
	                      int maximumPoolSize,
	                      long keepAliveTime,
	                      TimeUnit unit,
	                      BlockingQueue<Runnable> workQueue,
	                      ThreadFactory threadFactory,
	                      RejectedExecutionHandler handler) {
	//可以发现，corePoolSize最小允许为0，maximumPoolSize最小允许为1
	//且必须 maximumPoolSize >= corePoolSize
	if (corePoolSize < 0 ||
	        maximumPoolSize <= 0 ||
	        maximumPoolSize < corePoolSize ||
	        keepAliveTime < 0)
	    throw new IllegalArgumentException();
	// 必要的检测
	if (workQueue == null || threadFactory == null || handler == null)
	    throw new NullPointerException();

	this.acc = System.getSecurityManager() == null ?
	        null :
	        AccessController.getContext();

	this.corePoolSize = corePoolSize;
	this.maximumPoolSize = maximumPoolSize;
	this.workQueue = workQueue;
	this.keepAliveTime = unit.toNanos(keepAliveTime);  // 根据时间和单位，算出 ns的时间
	this.threadFactory = threadFactory;
	this.handler = handler;
}
```

+ `corePoolSize`和`maximumPoolSize`作为线程数量的限制。`corePoolSize`最小允许为0，`maximumPoolSize`最小允许为1。且必须 `maximumPoolSize >= corePoolSize`。
	+ 在`corePoolSize`范围内的工作线程可能得以一直不被回收，但工作线程的数量不会超过`maximumPoolSize`的。
+ `keepAliveTime`作为空闲线程等待超时时间。一般情况下，线程数量超过`corePoolSize`时，空闲线程超时后会被回收；但如果允许`CoreThreadTimeOut`，那么线程数量即使没超过`corePoolSize`（这些都是核心线程），空闲线程超时后也会被回收。
+ `workQueue`作为task队列，它是一个阻塞队列。我们把线程池的工作线程称为消费者，执行`线程池.execute()`的线程称为生产者。当消费者想取出元素而队列为空时，消费者可能会阻塞；当生产者想放入元素而队列已满时，生产者可能会阻塞。
+ `threadFactor`作为线程工厂，传入`Runnable`返回一个新`Thread`对象。有默认的`Executors.defaultThreadFactory`。
+ `handler`作为拒绝策略。当线程池状态为RUNNING时，如果工作线程数量已经达到`maxmumPoolSize`，且task队列已满，生产者会被拒绝；当线程池状态非RUNNING时，生产者肯定被拒绝。

### execute提交方法

生产者传递一个task给线程池，线程池负责异步地执行这个task，如果task既不能被马上执行，也不能入队，那么将执行拒绝策略。

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {//如果正在运行的线程数量 < corePoolSize
        //那么直接尝试把task交给一个新线程处理，不用先入队task了
        if (addWorker(command, true))//如果起新线程成功了，直接返回
            return;
        //执行到这里，说明情况发生了变化。
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {  // 如果线程池还在运行中，而且task入队成功
        int recheck = ctl.get();//入队成功后还需要再次检查
        if (! isRunning(recheck) && remove(command))//如果不在running状态，那么将task出队
            //执行拒绝策略
            reject(command);
        
        //1. 线程池是运行状态
        //2. 线程池不是运行状态，但task出队失败
        else if (workerCountOf(recheck) == 0)
            //因为corePoolSize允许为0，且有可能刚入队后，池里唯一的线程就死掉了
            addWorker(null, false);
    }
    //1. 如果线程池不在运行中
    //2. 如果线程池在运行中，但入队失败了
    else if (!addWorker(command, false))//尝试新起一个线程来处理task
        reject(command);
}
```

其实上面这个过程体现了`corePoolSize`、`maximumPoolSize`、task队列大小 之间的关系。我们把关键步骤拎出来，并忽略那些检测线程池非RUNNING状态的逻辑的话，那么只有这三步：

+ `addWorker(command, true)`。第二个实参代表如果线程数没有超过`corePoolSize`，那么新增一个线程直接执行task，自然，这个task不会被入队。
+ `workQueue.offer(command)`。将task入队，如果入队成功，说明没有超过队列大小。
+ `addWorker(command, false)`。第二个实参代表如果线程数没有超过`maximumPoolSize`，那么新增一个线程直接执行task，自然，这个task不会被入队。

这三步总是按照顺序执行的，根据`corePoolSize`、`maximumPoolSize`、task队列大小，这里体现了execute的执行策略：

+ 如果线程数量小于`corePoolSize`，那么不入队，尝试起一个新线程来执行task。
+ 如果线程数量大于等于了`corePoolSize`，那么只要队列未满，就将task入队。
+ 如果线程数量大于等于了`corePoolSize`，且队列已满（上一步入队失败）。那么以`maximumPoolSize`作为限制，再起一个新线程。——`addWorker(command, false)`成功。
+ 如果线程数量大于等于了`maximumPoolSize`，且队列已满。那么将执行拒绝策略。——`addWorker(command, false)`失败。

#### addWorker尝试新起线程

此函数尝试新起一个线程，其参数core的作用我们上面已经讲过了。

```java
private boolean addWorker(Runnable firstTask, boolean core) {
    //该循环在检测线程池状态的前提下，和线程数量限制的前提下，尝试增加线程数量
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // 如果状态已经不是running了
        if (rs >= SHUTDOWN &&
            // 三者都成立，括号内才返回true，括号外才返回false，函数才不会直接return
            ! (rs == SHUTDOWN &&//当前是SHUTDOWN状态
               firstTask == null &&//传入参数是null（非null说明是新task，但已经SHUTDOWN所以不能再添加）
               ! workQueue.isEmpty()))//队列非空
            //即有三种情况会造成addWorker直接false，不去新起线程了；还有一种特殊情况，addWorker会继续执行。
            //1. 如果当前是STOP以及以后的状态（肯定不需要新起线程了，因为task队列为空了）
            //2. 如果当前是SHUTDOWN，且firstTask参数不为null（非RUNNING状态线程池都不可以接受新task的）
            //3. 如果当前是SHUTDOWN，且firstTask参数为null，但队列空（既然队列空，那么也不需要新起线程）
         
            //下面的一种情况会继续执行
            //1. 如果当前是SHUTDOWN，且firstTask参数为null，且队列非空（特殊情况，需要新起线程把队列剩余task执行完）
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||//如果线程数量超过最大值
                wc >= (core ? corePoolSize : maximumPoolSize)) //如果线程数量超过特定值，根据core参数决定是哪个特定值
                return false;
            if (compareAndIncrementWorkerCount(c))//CAS尝试+1一次
                //如果CAS成功，说明这个外循环任务完成，退出大循环
                break retry;
            //CAS修改失败可能是：线程数量被并发修改了，或者线程池状态都变了
            c = ctl.get();
            if (runStateOf(c) != rs)//再次检查当前线程池状态，和当初保存的线程池状态
                continue retry;//如果改变，那么continue外循环，即重新检查线程池状态（毕竟线程池状态也在这个int里）
            // 如果只是 线程数量被并发修改了，那么接下来会继续内循环，再次CAS增加线程数量
        }
    }

    //此时，crl的线程数量已经成功加一，且线程池状态保持不变（相较于函数刚进入时）
    
    boolean workerStarted = false;//新线程是否已经开始运行
    boolean workerAdded = false;//新线程是否已经加入set
    Worker w = null;
    try {
        w = new Worker(firstTask);//构造器中利用线程工厂得到新线程对象
        final Thread t = w.thread;//获得这个新线程
        if (t != null) {//防止工厂没有创建出新线程
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();//加锁不是为了ctl，而是为了workers.add(w)
            try {
                // 需要再次检查状态
                int rs = runStateOf(ctl.get());
                if (rs < SHUTDOWN ||//如果线程还是运行状态
                    (rs == SHUTDOWN && firstTask == null)) {//如果线程是SHUTDOWN，且参数firstTask为null
                    if (t.isAlive()) //新线程肯定是还没有开始运行的
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {//加入集合成功，才启动新线程
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            //只要线程工厂创建线程成功，那么workerStarted为false只能是因为线程池状态发生变化，且现在一定不是running状态了
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

+ 想要新起线程，就必须将`ctl`的线程数量部分加1。
	+ 但这还有个大前提，那就是线程状态必须符合下面两种情况之一，才需要新起线程。
		+ 1.状态为RUNNING。
		+ 2.状态为SHUTDOWN，且task队列非空，这种情况需要新起线程来执行剩余task。当然，firstTask参数必须null，因为此时不允许处理新task了。并且如果firstTask参数为null，那么新起线程将会从task队列中取出一个来执行，这就达到了 新起线程来执行剩余task 的目的。

+ `ctl`的线程数量部分成功加1后，就需要创建新线程，并启动线程。
	+ 利用线程工厂创建Worker，Worker里包含了新线程。
	+ 检测线程池状态没有发生变化后（符合上面两种情况），将Worker加入集合，启动新线程。
	+ 检测线程池状态发生变化后，那新线程不能被启动，则需要把已创建的Worker从集合中移除，并且把ctl的线程数量部分再减1（其实就是addWorkerFailed）。

总之，此函数返回true就代表新线程成功启动了；返回false，就是因为当前线程池状态不对而没有启动新线程。

另外，在`new Worker(firstTask)`内部利用了线程工厂来创建的新Thread。

```java
Worker(Runnable firstTask) {
    setState(-1); // inhibit interrupts until runWorker
    this.firstTask = firstTask;
    this.thread = getThreadFactory().newThread(this);//this就是 正在完成初始化的Worker对象自己
}
```

#### addWorkerFailed回滚

此函数把已创建的Worker从集合中移除（如果存在的话），并且把`ctl`的线程数量部分再减1。不过前面也说过，调用`addWorkerFailed`的前提很可能是线程池状态发生了变化，所以这里需要`tryTerminate`。

```java
private void addWorkerFailed(Worker w) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        if (w != null)
            workers.remove(w);
        decrementWorkerCount();
        tryTerminate();
    } finally {
        mainLock.unlock();
    }
}
```

#### tryTerminate 帮助Terminate

此函数是用来帮助线程池终止的，将状态更新为TIDYING，最后更新为TERMINATED。

可以帮助的情况为：

+ 当前SHUTDOWN，且线程数量和任务队列大小都为0。
+ 当前STOP（隐含了任务队列大小为0），且线程数量为0。

```java
final void tryTerminate() {
    for (;;) {
        int c = ctl.get();
        if (isRunning(c) ||//当前还是Running，那么根本不应该tryTerminate
            runStateAtLeast(c, TIDYING) ||//已经是TIDYING，那么不需要当前线程去帮忙了，别的线程会帮忙变成TERMINATED
            (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))//虽然SHUTDOWN，但任务队列非空
            return;
        
         //1. 状态为SHUTDOWN，任务队列为空，继续尝试
         //2. 状态为STOP
        
         //如果线程数量都不是0，那么肯定不能tryTerminate，直接返回
        if (workerCountOf(c) != 0) {
            //中断一个空闲线程，中断状态会自己传播
            interruptIdleWorkers(ONLY_ONE);
            return;
        }
        

        //说明线程数量确实为0。现在已经满足可以帮助的条件
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            //CAS更新为TIDYING状态
            if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                try {
                    terminated();//空实现
                } finally {
                    //谁变成了TIDYING，谁就负责变成TERMINATED（这解释了前面遇到TIDYING的情况）
                    ctl.set(ctlOf(TERMINATED, 0));
                    termination.signalAll();//已经获得了锁，现在可以唤醒条件队列的线程了
                }
                return;
            }
        } finally {
            mainLock.unlock();
        }
        // ctl.compareAndSet(c, ctlOf(TIDYING, 0))如果失败走这里，肯定是因为别的线程
		// 把状态变成了TIDYING状态。下一次循环会直接退出
    }
}
```

此函数检测到可以帮忙时，就把当前状态变成TIDYING，执行完`terminated()`后，再把状态变成TERMINATED。

#### interruptIdleWorkers

此函数尝试中断空闲的Worker线程。Worker线程在执行task的前提是持有自己的Worker锁，相反，空闲的线程是没有持有自己的Worker锁的，所以当前线程执行`w.tryLock()`是能返回true的。参数`onlyOne`为true时，只中断一个空闲的Worker线程。

该方法的作用是使在等待task的线程中断，使它们退出阻塞并检查状态、各种参数是否发生了变化（它们还是可能继续等待）。

```java
private void interruptIdleWorkers(boolean onlyOne) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (Worker w : workers) {
            Thread t = w.thread;
            if (!t.isInterrupted() && w.tryLock()) {
                //到这一步需要符合以下两种情况
                //1、线程t没有被中断过
                //2、worker w是空闲的，因为worker执行task的前提是持有自己的worker锁，因此w.tryLock返回true表示idle
                //注意，tryLock一定要放在右边，不然可能获得锁却不释放锁
                try {
                    t.interrupt();
                } catch (SecurityException ignore) {
                } finally {
                    w.unlock();
                }
            }
            if (onlyOne)
                break;
        }
    } finally {
        mainLock.unlock();
    }
}
```

我们执行这句`t.interrupt()`，就中断了Worker线程。但Worker线程是怎么做到“只中断一个，就传播这个中断状态”的，我们还得看Worker的实现。

### Worker

```java
private final class Worker
    extends AbstractQueuedSynchronizer
    implements Runnable
{
    private static final long serialVersionUID = 6138294804551838833L;
    //worker在哪个线程中执行
    final Thread thread;
    //初始任务
    Runnable firstTask;
    //已完成的task的数量
    volatile long completedTasks;
    ......
}
```

Worker的首要目的是保存线程对象和这个工作线程的第一个task。从下面的构造器可以看到

```java
Worker(Runnable firstTask) {
    setState(-1); // inhibit interrupts until runWorker
    this.firstTask = firstTask;
    this.thread = getThreadFactory().newThread(this);
}
```

#### 对AQS的实现

可以看到Worker继承了AQS，目的是为了使用AQS的同步队列。继承了就需要实现AQS的抽象方法。

```java
private final class Worker
    extends AbstractQueuedSynchronizer
    implements Runnable
{
    private static final long serialVersionUID = 6138294804551838833L;
    final Thread thread;
    Runnable firstTask;
    volatile long completedTasks;

    Worker(Runnable firstTask) {
        setState(-1); 
        this.firstTask = firstTask;
        this.thread = getThreadFactory().newThread(this);
    }
    
    public void run() {
        runWorker(this);
    }

    //state非0表示锁被持有了
    protected boolean isHeldExclusively() {
        return getState() != 0;
    }

    //尝试获取锁。CAS修改state:0->1 表示尝试获取锁成功，并设置独占线程
    protected boolean tryAcquire(int unused) {
        if (compareAndSetState(0, 1)) {
            setExclusiveOwnerThread(Thread.currentThread());
            return true;
        }
        return false;
    }

    //释放锁，state->0
    protected boolean tryRelease(int unused) {
        setExclusiveOwnerThread(null);
        setState(0);
        return true;
    }

    public void lock()        { acquire(1); }
    public boolean tryLock()  { return tryAcquire(1); }
    public void unlock()      { release(1); }
    public boolean isLocked() { return isHeldExclusively(); }

    // 如果工作线程已经执行过它人生中第一个task了(那么state就肯定不是-1了)
    // 那么就中断它
    void interruptIfStarted() {
        Thread t;
        if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
            try {
                t.interrupt();
            } catch (SecurityException ignore) {
            }
        }
    }
}
```

- 仅仅为了使用AQS的独占锁部分。利用独占锁的互斥，可以工作线程在从未开始时（state为-1）和正在执行task期间（state为1）不会被中断。
- 对`tryAcquire`的实现是不可重入的，原因之后再讲。总之，为了避免因不可重入而无限阻塞，只要避免当前线程持有锁的情况再去`acquire(1)`，就不会出现无限阻塞。
	- 相反，`tryAcquire`总会是安全的。

#### 对Runnable的实现

把`addWorker`里面启动线程的那几句单独拎出来：

```java
 w = new Worker(firstTask);
final Thread t = w.thread;
t.start();
```

假设线程工厂用的就是默认的`Executors.defaultThreadFactory`：

```java
        Worker(Runnable firstTask) {
            setState(-1); // inhibit interrupts until runWorker
            this.firstTask = firstTask;
            this.thread = getThreadFactory().newThread(this);//this就是还未初始化完成的这个Worker对象
        }
//Executors.java
    static class DefaultThreadFactory implements ThreadFactory {
        public Thread newThread(Runnable r) {//这里传的是Worker实例
            Thread t = new Thread(group, r,
                                  namePrefix + threadNumber.getAndIncrement(),
                                  0);
            return t;
        }
    }
```

好了，现在`t.start()`启动的线程终于找到了，是上面这个`DefaultThreadFactory#newThread`返回的Thread实例，而这个Thread实例持有一个`Worker`实例作为它的`Runnable target`成员。所以当调用`t.start()`的过程是如下这样的：

```java
//Thread
    public void run() {
        if (target != null) {//target是Worker实例
            target.run();
        }
    }
//ThreadPoolExecutor.Worker
        public void run() {
            runWorker(this);
        }
```

总之就是套了个壳子，最后还是执行到了Worker的run方法这里，而它又调用了`runWorker`。

#### runWorker

`runWorker`是线程池工作线程的全部运行逻辑，一定是工作线程来执行`runWorker`的。

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();//获得当前线程对象
    Runnable task = w.firstTask;//获得第一个task（这里可能为null）
    w.firstTask = null;//释放引用
    w.unlock(); 
    // 此时state为0，Worker锁此时可以被抢。且此时工作线程可以被中断了
    //还有一个作用，就是state的初始值为-1，需要将state置为0
    boolean completedAbruptly = true;
    try {
        //1. 如果第一个task不为null，开始执行循环
        //2. 如果第一个task为null，但从task队列里获得的task不为null，也开始循环
        //3. 如果task队列获得到的也是null，那么结束循环
        while (task != null || (task = getTask()) != null) {
            w.lock();//执行task前，先获得锁
            // 第一个表达式 (runStateAtLeast(ctl.get(), STOP) || (Thread.interrupted() && runStateAtLeast(ctl.get(), STOP)))
			// 无论线程池状态是什么都会消耗掉当前线程的中断状态（如果当前线程确实被中断了），
			// 并且只有线程池状态是STOP的情况下，且当前线程被中断了，才会返回true。
			// 第二个表达式 !wt.isInterrupted()
			// 因为第一个表达式永远会消耗掉中断状态，所以第二个表达式肯定为true
            // 总之，重点在第一个表达式。

            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                // 1. 如果线程池状态为STOP，且当前线程被中断，马上恢复中断状态
                // 2. 如果线程池状态为其他，且当前线程被中断，仅仅消耗掉这个中断状态，不进入分支
                wt.interrupt();//恢复中断状态
            try {
                //空实现的方法，如果抛出异常，completedAbruptly将为true
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    task.run();//执行task
                } catch (RuntimeException x) {
                    thrown = x; throw x;//这里抛出异常，completedAbruptly也将为true
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    //空实现方法
                    afterExecute(task, thrown);//task.run()抛出异常的话，至少afterExecute可以执行
                }
            } finally {//无论上面在beforeExecute或task.run()中抛出异常与否，最终都会执行这里
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;//上面循环是正常结束而没有抛出异常，这个变量才为false
    } finally {
        //无论循环正常结束(getTask返回null，completedAbruptly为false)，
		//还是循环中抛出了异常(completedAbruptly为true)，都会执行这句。
		//代表当前Worker马上要寿终正寝了
        processWorkerExit(w, completedAbruptly);
    }
}
```

- 进入循环前，将state设置为0，使得Worker锁可以被抢。
- 循环是工作线程的主要逻辑，每次循环通过条件里的`getTask()`获得task来执行。当然，`getTask()`必然是可以阻塞等待直到从队列取得元素的。
	- 执行task前，必须先消耗中断状态（如果线程已被中断），因为中断状态不清理会导致`getTask()`里无法阻塞。并且只有在线程池状态为STOP时（task队列已空）且线程已被中断，才恢复线程的中断状态（这看起来可以用来保证，在当前循环执行task后下一次循环getTask()会抛出中断异常，但实际上getTask()发现STOP状态会直接返回null；当然还有一种可能，就是task.run()会检测中断状态抛出中断异常）。
	- 执行task前，先执行`beforeExecute`。如果抛出异常，会导致`task.run()`不执行。
	- 执行task时（`task.run()`），可能抛出异常。
	- 无论执行task时是否抛出异常，都会执行`afterExecute`。
	- 每次循环结束前，无论前面有没有抛出异常，都会清空一些变量，并释放Worker锁，因为这次拿到的task已经执行完毕。
- 从循环结束有两个原因：1. `task.run()`返回null了，循环正常结束（`completedAbruptly`为false）。 2. 在执行task时抛出了异常，也会结束循环（`completedAbruptly`为true）。无论哪种情况，当前Worker线程都会马上寿终正寝了。

简单的说，`runWorker`做的就是每次循环中从队列中取出一个task来执行，如果队列为空，那么阻塞等待直到队列非空取到task。这就是每个工作线程的工作内容。

#### getTask

此函数尝试从task队列取得一个task，注意在`getTask`执行时，当前worker线程是没有持有自己的worker锁的，所以在`getTask`中支持被中断。

```java
private Runnable getTask() {
    boolean timedOut = false;  //保存超时poll操作是否操作，

    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // 这里条件是一个组合关系：
		// 1. rs >= SHUTDOWN 和 workQueue.isEmpty() 说明没有task可以获得了
		// 2. rs >= SHUTDOWN 和 rs >= STOP（隐式的说明 task队列为空）
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();//循环CAS，直到线程数量减一
            return null;//返回null代表取task失败
        }

        int wc = workerCountOf(c);

        // 这个变量控制，出队操作是否需要为超时版本。
		// 1. 如果allowCoreThreadTimeOut为true，那么所有线程都会使用超时版本的出队操作
		// 2. 如果wc > corePoolSize，那么超过部分的线程应该使用超时版本的出队操作
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        // 第一个表达式((wc > maximumPoolSize || (timed && timedOut))
        // 如果wc大于了最大线程数（因为动态调小了maximumPoolSize）或超时操作已经超时，
        // 这说明当前worker很可能需要抛弃。
        // 第二个表达式(wc > 1 || workQueue.isEmpty())
        // 如果线程数至少为2（除了当前worker线程外还有其他线程），
        // 或者线程数为1（只有当前worker线程）但队列为空：
        // 这说明不用担心，队列剩余task没有线程去取，即确定了当前worker需要抛弃
        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))//如果CAS尝试减一成功
                return null;//直接返回null
            continue;//如果CAS失败，可能有多个线程同时修改了ctl的线程数。
            //而这个分支是否还会进入需要下一个循环再判断
        }

        try {
            Runnable r = timed ?//根据变量来判断，使用超时poll，还是take
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) ://限时阻塞
                workQueue.take();//无限阻塞
            if (r != null)//成功获得了task
                return r;
             //只有限时阻塞才可能返回null，且肯定是超时操作超时了
            timedOut = true;
        } catch (InterruptedException retry) {
            //无论上面是哪种操作，在阻塞期间被中断的话，当前worker线程会抛出中断异常
            timedOut = false;//不算超时，所以这里设为false
            //继续下一次循环，下一次可能会返回null
        }
    }
}
```

+ 在获取队列元素之前，必须要判断线程池状态和其他一些情况。视情况而定，可能会直接返回null，代表获取失败或者说不应该由当前线程来获取。
	+ 如果线程池状态为SHUTDOWN且队列为空，那么没有task可以让当前线程获得，返回null。
	+ 如果线程池状态为STOP（隐含队列为空），那么没有task可以让当前线程获得，返回null。
	+ 如果不是上面的情况，那么只是说明有task可以取。但还得继续判断情况，如果同时满足以下两个情况则则不可以去取task（返回null）：
		+ 1.如果当前线程数量已经超过`maximumPoolSize`（这是因为动态调小了`maximumPoolSize`），或者虽然没有超过但上一次的超时poll动作已经超时了（做超时操作的前提是`allowCoreThreadTimeOut || wc > corePoolSize`，既然超时了，当前线程也就不用去取task了）。满足的话，说明当前线程应该是需要放弃取task的，但还得满足下一个情况。
		+ 2.因为即将放弃取task，所以得防止“队列里有task但是没有工作线程在工作”的情况。在队列非空时，除了当前线程还必须有别的线程，毕竟当前线程马上要放弃了。
		+ 满足了上面两种情况，则当前线程要放弃取task了，但在结束`getTask`之前要把ctl的线程数量减一。
		+ 但CAS修改ctl的线程数量可能失败，失败后再次走上面的流程。完全有可能，失败后再走上面流程就不会放弃了，比如当前线程数量为corePoolSize+1（考虑allowCoreThreadTimeOut为false），有两个工作线程都超时了，第一个线程放弃并修改线程数量成功，第二个线程也想放弃但修改ctl失败下一次循环发现wc > corePoolSize不成立，也就不放弃了，继续去做取task动作。
+ 上面这些判断都通过了，说明当前线程确实需要取得一个task。
	+ 根据`timed`变量做 限时阻塞的出队动作、或无限阻塞的出队动作。如果成功出队一个task，则返回它。

总之，`getTask`返回非null值代表当前worker线程应该继续工作；返回null值代表当前条件下获取task失败，当前条件是考虑了线程池状态和当前线程状态（是否超过核心线程数，是否限时阻塞已经超时）。

#### processWorkerExit

当前线程已经从`runWorker`中的循环中退出，可能是因为`getTask`返回null，也可能是执行task时抛出了异常。总之，当前worker线程已经马上要寿终正寝了，所以来调用`processWorkerExit`。

```java
//传入Worker已经马上要寿终正寝
private void processWorkerExit(Worker w, boolean completedAbruptly) {
    if (completedAbruptly) // 如果runWorker中的循环不是正常结束的，则需要此函数来减小线程数
        decrementWorkerCount(); // 否则是runWorker里的getTask做了这件事（getTask返回了null）

    //此时，ctl的线程数量是不包括传入的Worker了
    
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        //传入Worker的完成任务数量 加到 线程池的总和上去
        completedTaskCount += w.completedTasks;
        workers.remove(w);
    } finally {
        mainLock.unlock();
    }

    tryTerminate();//帮助Terminate

    int c = ctl.get();
    if (runStateLessThan(c, STOP)) {//如果状态为RUNNING或SHUTDOWN，才可能需要补充线程
        //但补充前还需要判断一下
        if (!completedAbruptly) {//如果传入Worker是正常结束的
            int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
            if (min == 0 && ! workQueue.isEmpty())//如果为0且队列非空，那么需要再加1
                min = 1;
            if (workerCountOf(c) >= min)//如果当前线程数量满足要求
                return;// 那么不需要补充线程，直接返回
        }
        // 1. 如果传入Worker是因为抛出异常而结束，那么肯定补充
		// 2. 如果传入Worker是正常结束，那么视需要而补充
        addWorker(null, false);
    }
}
```

前面做了一些收尾工作，比如ctl减一，workers移除。当前worker即将要寿终正寝，但可能需要在结束前补充一个worker。

- 如果当前worker是因为抛出异常从而结束自己生命的，那么肯定补充（`!completedAbruptly`为false）。
- 如果当前worker是因为`getTask`返回null从而结束自己生命的，那么在当前线程数量不够最小线程数时，才会去补充。

这也解释了`interruptIdleWorkers(ONLY_ONE)`为什么会传播中断状态。因为一个空闲的工作线程被中断后，会去执行`processWorkerExit`里的`tryTerminate`，而`tryTerminate`里又会去调用`interruptIdleWorkers(ONLY_ONE)`唤醒另一个空闲线程。

#### 独占锁的作用

我们知道在`runWorker`中，如果worker线程还没有开始，或者正在执行task，state一定是非0的，这就使得别的线程无法获得worker锁。这样别的线程在调用`interruptIdleWorkers`时，是无法中断正在执行task的worker线程的。

而独占锁被设计为不可重入的原因是为了防止自己中断自己。
比如生产者传入的task是这样实现的：

```java
class MyTask implements Runnable {
    @Override
    public void run() {
        threadPoolExecutor.setCorePoolSize(10);
    }
}
```

而`setCorePoolSize`里又有可能调用到`interruptIdleWorkers`，所以不可重入就防止了自己中断自己。

### 核心线程预启动

初始化线程池后，如果没有生产者来调用`execute`之类的方法的话，是肯定不会启动任何线程的。所以，如果构造线程池时给的task队列非空，那么直到第一次调用`execute`之前，都不会有任何一个工作线程。

```java
public boolean prestartCoreThread() {
    return workerCountOf(ctl.get()) < corePoolSize &&
        addWorker(null, true);
}

public int prestartAllCoreThreads() {
    int n = 0;
    while (addWorker(null, true))
        ++n;
    return n;
}
```

### 关闭线程池

#### shutdown

```java
private void advanceRunState(int targetState) {
    for (;;) {
        int c = ctl.get();
        if (runStateAtLeast(c, targetState) ||
            ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c))))//此函数会保持之前的线程数量
            break;
    }
}

public void shutdown() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();//检查权限
        advanceRunState(SHUTDOWN);//改变线程池状态
        interruptIdleWorkers();//中断所有空闲线程
        onShutdown();  // 钩子方法，空实现
    } finally {
        mainLock.unlock();
    }
    tryTerminate();//尝试Terminate
}
```

- 线程池状态变成SHUTDOWN后，就无法再提交新task给线程池了。
- `interruptIdleWorkers`可以中断那些阻塞在`workQueue.超时poll`或`workQueue.take`上的线程，它们被中断后，会检查改变后的状态和各种参数以做相应的处理。
- 有可能此时已经满足了Terminate的条件，所以必须尝试一下`tryTerminate`。

#### shutdownNow

```java
public List<Runnable> shutdownNow() {
    List<Runnable> tasks;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
        advanceRunState(STOP);//改变线程池状态
        interruptWorkers();//中断所有线程
        tasks = drainQueue();//清空队列
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
    return tasks;
}
```

+ 线程池状态变成STOP后，同样无法再提交新task给线程池了。
+ `interruptWorkers`中断不仅中断空闲线程，正在工作的线程也会中断，注意这无视了工作线程已经持有的worker锁。但如果工作线程的执行task不关心中断的话，那么也没有意义。
+ `tasks = drainQueue()`清空队列。

### awaitTermination

此函数可以让当前线程一直阻塞直到线程池所有任务都结束、或者超时。

```java
public boolean awaitTermination(long timeout, TimeUnit unit)
    throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (;;) {
            if (runStateAtLeast(ctl.get(), TERMINATED))
                return true;
            if (nanos <= 0)//如果发现剩余时间少于0，说明超时
                return false;
            nanos = termination.awaitNanos(nanos);//限时阻塞，阻塞在这个termination条件队列上
        }
    } finally {
        mainLock.unlock();
    }
}
```

总之，就是利用了AQS的条件队列，让等待Termination的线程都阻塞在条件队列上。当然，在`tryTerminate`中，会执行`termination.signalAll()`来唤醒条件队列上的所有线程。

### 调整线程池参数

#### setCorePoolSize

此函数设置新的`corePoolSize`，相较之前如果发生了变化，则需要多退少补。

+ `corePoolSize`变小，则需要停止掉一些空闲的核心线程。

- `corePoolSize`变大，则需要补充一些新的核心线程，但如果发现队列为空就不需要再增加了。

```java
public void setCorePoolSize(int corePoolSize) {
    if (corePoolSize < 0)
        throw new IllegalArgumentException();
    int delta = corePoolSize - this.corePoolSize;
    this.corePoolSize = corePoolSize;
    if (workerCountOf(ctl.get()) > corePoolSize)
        interruptIdleWorkers();//中断所有空闲的线程，使它们检查更新后的参数做出相应的处理
    else if (delta > 0) {
        int k = Math.min(delta, workQueue.size());
        while (k-- > 0 && addWorker(null, true)) {
            if (workQueue.isEmpty())
                break;
        }
    }
}
```

如果`corePoolSize`调高了，那么将当前工作线程数增加，直到增加的数量等于等待队列中的task的数量或者直到总数量达到新的`corePoolSize`。

#### setMaximumPoolSize

```java
public void setMaximumPoolSize(int maximumPoolSize) {
    if (maximumPoolSize <= 0 || maximumPoolSize < corePoolSize)
        throw new IllegalArgumentException();
    this.maximumPoolSize = maximumPoolSize;
    //在调低的情况下，如果当前线程数量多于新的maximumPoolSize，那么中断空闲的线程
    if (workerCountOf(ctl.get()) > maximumPoolSize)
        interruptIdleWorkers();
}
```

#### setKeepAliveTime

此函数设置新的keepAliveTime。如果新的keepAliveTime更小，那么中断空闲线程，让这些线程使用新的`keepAliveTime`来调用超时`poll`。

```java
public void setKeepAliveTime(long time, TimeUnit unit) {
    if (time < 0)
        throw new IllegalArgumentException();
    if (time == 0 && allowsCoreThreadTimeOut())//不可兼得
        throw new IllegalArgumentException("Core threads must have nonzero keep alive times");
    long keepAliveTime = unit.toNanos(time);
    long delta = keepAliveTime - this.keepAliveTime;
    this.keepAliveTime = keepAliveTime;
    if (delta < 0)
        interruptIdleWorkers();
}
```

- `KeepAliveTime=0`和`allowsCoreThreadTimeOut=true`不可兼得。
- 如果`KeepAliveTime=0`（allowsCoreThreadTimeOut肯定为false），会造成核心数量以外的线程在发现队列为空时，就马上结束自己。

#### allowCoreThreadTimeOut

此函数设置`allowCoreThreadTimeOut`，这个成员为true，代表允许核心线程也会因为超时得不到task而结束自己。当从false变成true，则需要中断那些空闲的核心线程。

```java
public void allowCoreThreadTimeOut(boolean value) {
    if (value && keepAliveTime <= 0)
        throw new IllegalArgumentException("Core threads must have nonzero keep alive times");
    if (value != allowCoreThreadTimeOut) {
        allowCoreThreadTimeOut = value;
        if (value)
            interruptIdleWorkers();
    }
}
```

#### setThreadFactory

设置线程工厂。

```java
public void setThreadFactory(ThreadFactory threadFactory) {
    if (threadFactory == null)
        throw new NullPointerException();
    this.threadFactory = threadFactory;
}
```

#### setRejectedExecutionHandler

设置拒绝策略。

```java
public void setRejectedExecutionHandler(RejectedExecutionHandler handler) {
    if (handler == null)
        throw new NullPointerException();
    this.handler = handler;
}
```

### 阻塞队列的选择

阻塞队列一旦在构造线程池给定后，就无法改变了。选择不同的阻塞队列可能会使得线程池的表现完全不一样。

![](../../../pic/42)

#### Direct handoffs

这种队列提供一种生产者直接提交task给消费者的功能，比如`SynchronousQueue`。

+ 生产者调用会`ThreadPoolExecutor.execute(command) -> workQueue.offer(command)->SynchronousQueue.transfer(command, true, 0)`，最终调用的是超时机制的offer，超时的时间为0，即立即。offer有两种表现：1. 已有消费者在等待，那么返回true，代表直接转交task给消费者 2. 没有消费者在等待，那么直接返回false，代表转交失败。这也说明线程池的`SynchronousQueue`内只可能有request node（`getTask()`的空闲线程）。
+ 由上一条可知，工作线程数量达到核心线程数时，再提交task就会马上创建新线程，前提是满足`maximumPoolSize`的限制。所以看来，`SynchronousQueue`是根本起不到缓存task的作用的。

#### Unbounded queues

无界队列，比如`LinkedBlockingQueue`。此时`maximumPoolSizes`不会起到作用，因为当线程数量已经达到核心线程数时，再调用`workQueue.offer(command)`肯定会入队成功返回true。

- 如果消费者处理task太慢，会导致task队列无限增长（无限缓存task），可能导致内存不足。
- `LinkedBlockingQueue`也可以设置容量，最好进行设置。
- `LinkedBlockingQueue`内部使用两个锁分别控制入队和出队，效率相对更高。

#### Bounded queues

有界队列，比如`ArrayBlockingQueue`。此时`ThreadPoolExecutor`的`execute`方法的策略能起到作用。

- 入队和出队时竞争的是同一个锁，并发效率比`LinkedBlockingQueue`低。

### 拒绝策略

两种情况消费者来提交task会被拒绝：

- 工作线程数达到了`maxmumPoolSize`，且task队列已满。
- 线程池正在关闭，或已经关闭。

```java
final void reject(Runnable command) {
        handler.rejectedExecution(command, this);
}
```

#### AbortPolicy 抛出异常

```java
//默认的拒绝策略
private static final RejectedExecutionHandler defaultHandler = new AbortPolicy();

public static class AbortPolicy implements RejectedExecutionHandler {

    public AbortPolicy() {
    }

    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        throw new RejectedExecutionException("Task " + r.toString() +//总是抛出异常
                " rejected from " +
                e.toString());
    }
}
```

#### CallerRunsPolicy 调用者执行

只要线程池还处于RUNNING状态，就让调用者同步执行这个task。否则，直接不给任何提示，放弃执行这个task。

```java
public static class CallerRunsPolicy implements RejectedExecutionHandler {

    public CallerRunsPolicy() {
    }

    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        //只要线程池还处于RUNNING状态
        if (!e.isShutdown()) {
            //让调用者同步执行这个task
            r.run();
        }
    }
}
```

#### DiscardOldestPolicy 丢弃最老任务

- 如果线程池已经处于非RUNNING状态，不给任何提示，放弃执行这个task。
- 如果是RUNNING状态，那么出队那个入队最久的task即队头task，然后把参数`Runnable r`再次提交给线程池，但也不一定再次提交就不会被拒绝。

```java
public static class DiscardOldestPolicy implements RejectedExecutionHandler {

    public DiscardOldestPolicy() {
    }

    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            //出队那个入队最久的task即队头task
            e.getQueue().poll();
            //再次提交任务
            e.execute(r);
        }
    }
}
```

#### DiscardPolicy 丢弃任务

什么也不做。

```java
public static class DiscardPolicy implements RejectedExecutionHandler {

        public DiscardPolicy() { }

        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        }
}
```

#### 自定义

实现`RejectedExecutionHandler`接口即可。

```java
public interface RejectedExecutionHandler {
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}
```

### 内置线程池

在Executors.java中提供一些静态方法，返回一些默认配置的线程池。

#### newFixedThreadPool

```java
 public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
}
```

- 因为是无界队列，所以`corePoolSize`不会起到作用，工作线程不会超过`corePoolSize`。就算不考虑前面一点，`corePoolSize`与`maximumPoolSize`相等，线程数量也不会超过`nThreads`的。
- 这意味着线程数量会一直增长到nThreads，数量达到nThreads后不会再增加，直到线程池关闭，这nThreads个线程都会一直保持运行。
- 时间参数不会起到作用，因为这只对核心数量以外的线程起作用。
- 但`newFixedThreadPool`返回的线程池可以设置改变其线程池参数。

#### newSingleThreadExecutor

```java
 public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
}
```

- 分析同上，这个线程池只存在一个工作线程。
- 但`newSingleThreadExecutor`的返回实例是`FinalizableDelegatedExecutorService`，它与`ThreadPoolExecutor`没有继承关系，所以你无法改变其参数。

#### newCachedThreadPool

```java
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
}
```

+ 先看看给这个线程池提交一个task后的过程：
	+ 生产者执行`execute`，由于核心线程数为0，先入队`workQueue.offer(command)`返回false，然后`addWorker(null, false)`启动一个新的工作线程。
	+ 工作线程开始执行run函数，开始执行第一个task，执行完后限时阻塞在了`task = getTask()`（由于核心线程数为0，每个线程都会检测空闲时间），60秒后`getTask`返回null，当前线程马上结束，执行`processWorkerExit`。
	+ 执行`processWorkerExit`期间，发现线程数量是0，需要补充一个工作线程，执行`addWorker(null, false)`。
	+ 新起的工作线程又会重复上面的步骤。
	+ 总之，`newCachedThreadPool`返回的线程池在没有task时，会始终保持一个活着的工作线程。
+ 对于所有工作线程来说，只要空闲了60秒，就结束自己。但线程池会始终保持一个活着的工作线程。
	+ 空闲的工作线程如果在60秒内等到了task，那么就能复用这个工作线程。
+ 对于生产者来说，提交task要么是直接提交给了空闲的工作线程，要么是起一个线程（因为`maximumPoolSize`为`Integer.MAX_VALUE`）。

### 总结

+ 理解`ThreadPoolExecutor`的关键在于理解`execute`提交task的执行逻辑：
	+ `addWorker(command, true)`。在线程数没有达到`corePoolSize`时，新起一个线程来执行task。核心线程数量内的线程不会被回收。
	+ `workQueue.offer(command)`。在线程数达到`corePoolSize`后，则不起线程，而是先入队task。
	+ `addWorker(command, false)`。如果线程数达到`corePoolSize`且队列已满（更严谨点，应该是入队失败），则新起一个线程来执行task。但这个线程可能会因为空闲地太久而被回收。
+ 阻塞队列的选择也会影响到`execute`的逻辑。
	+ 有界队列，使得`execute`的策略可以正常运行。
	+ 无界队列，使得`maximumPoolSize`失去作用。
	+ 直接提交队列，使得队列失去缓存的作用。
+ 线程池关闭后，肯定无法提交新task了。
	+ 如果执行的是`shutdown`，每个工作线程完成当前task后还会去执行队列中的剩余task。
	+ 如果执行的是`shutdownNow`，队列中剩余task会被清空，每个工作线程完成当前task后，线程池就会结束使命。
+ Worker是工作线程的实现，它继承了AQS实现了独占锁部分，目的是为了让工作线程在未开始执行task或正在执行task时，不会被`interruptIdleWorkers`中断。
+ 工作线程执行task期间如果抛出了异常，一定会补充新的工作线程。

