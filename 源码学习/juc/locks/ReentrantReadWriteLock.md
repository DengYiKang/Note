# ReentrantReadWriteLock

## 源码

### 成员

```java
private static final long serialVersionUID = -6992448646407690164L;
//读锁
private final ReentrantReadWriteLock.ReadLock readerLock;
//写锁
private final ReentrantReadWriteLock.WriteLock writerLock;
//AQS子类
final Sync sync;

abstract static class Sync extends AbstractQueuedSynchronizer{...}

static final class NonfairSync extends Sync{...}

static final class FairSync extends Sync{...}

public static class ReadLock implements Lock, java.io.Serializable {...}

public static class WriteLock implements Lock, java.io.Serializable{...}
```

ReentrantReadWriteLock定义了两个内部类ReadLock和WriteLock，同时拥有读写锁分别一个。读写锁公用一个AQS子类sync。

### 构造器

```java
public ReentrantReadWriteLock() {
    this(false);
}

public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}
//ReadLock这里只贴出构造器代码，其他省略
public static class ReadLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = -5992448646407690164L;
    private final Sync sync;

    protected ReadLock(ReentrantReadWriteLock lock) {
        sync = lock.sync;
    }
    
}
//WriteLock这里只贴出构造器代码，其他省略
public static class WriteLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = -4992448646407690164L;
    private final Sync sync;

    protected WriteLock(ReentrantReadWriteLock lock) {
        sync = lock.sync;
    }

}
```

可以发现，ReentrantReadWriteLock的默认模式是非公平的，读锁与写锁公用一个AQS子类sync。

### sync

#### 构造器以及成员变量

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 6317671515068378041L;
		
    	//偏移量，state为int类型，高16位为读锁的拥有数量，低16位为写锁的拥有数量
        static final int SHARED_SHIFT   = 16;
    	//单位1，对于读锁而言。
        static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
    	//读写锁上限
        static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
    	//写锁的掩码，state&EXCLUSIVE_MASK
        static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
		
    	//获得读锁的拥有量
        static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
    	//获得写锁的拥有量
        static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }

    	//记录各个线程的读锁的拥有数量，与ThreadLocal相结合作为线程私有变量
    	//写锁不需要记录，因为只能有一个线程持有写锁，至于重入数量，可以直接查看state
        static final class HoldCounter {
            int count = 0;
            //注意这里引用的是id，而不是thread的引用，避免影响gc
            final long tid = getThreadId(Thread.currentThread());
        }

    	//将HoldCounter作为线程私有的
        static final class ThreadLocalHoldCounter
            extends ThreadLocal<HoldCounter> {
            public HoldCounter initialValue() {
                return new HoldCounter();
            }
        }
    	//ThreadLocal子类的引用，用于使用对各个线程的私有变量HoldCounter的get/set方法
        private transient ThreadLocalHoldCounter readHolds;
    	//做缓存用，提高速度
        private transient HoldCounter cachedHoldCounter;
		//这两个跟cachedHoldCounter一样的作用，做缓存用
        private transient Thread firstReader = null;
        private transient int firstReaderHoldCount;

        Sync() {
            readHolds = new ThreadLocalHoldCounter();
            setState(getState()); 
        }
}
```

#### 辅助方法

```java
//尝试获取锁时，判断是否需要阻塞
//在非公平锁的实现中，总是返回true
//在公平锁的实现中，如果是队首就返回true，否则返回false
abstract boolean readerShouldBlock();
abstract boolean writerShouldBlock();
```

#### 写锁

```java
//尝试释放写锁
//如果当前线程未持有写锁，那么抛出异常
//释放后如果写锁的state为0，那么将exclusiveOwnerThread置空
protected final boolean tryRelease(int releases) {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    int nextc = getState() - releases;
    boolean free = exclusiveCount(nextc) == 0;
    if (free)
        setExclusiveOwnerThread(null);
    setState(nextc);
    return free;
}

//尝试获取锁
//1、读锁数量必须为0
//2、写锁可以非0，但是必须保证当前线程已经持有写锁（重入的情况）
//3、如果获取后写锁数量溢出，那么抛出异常
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    int c = getState();
    int w = exclusiveCount(c);
    if (c != 0) {
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        setState(c + acquires);
        return true;
    }
    //writerShouldBlock封装了公平或非公平的信息
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```

```java
//除了没有调用writerShouldBlock外，效果和tryAcquire相同
final boolean tryWriteLock() {
    Thread current = Thread.currentThread();
    int c = getState();
    if (c != 0) {
        int w = exclusiveCount(c);
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
    }
    if (!compareAndSetState(c, c + 1))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```

#### 读锁

```java
//unused表示参数不会被使用到，相当于无参方法
//如果释放后读写锁的数量都为0，那么返回true；否则返回false
//注意这个返回值并不是表示是否release成功的
//只要不抛出异常，该方法返回就表示release成功
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    if (firstReader == current) {
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        //查看缓存的HoldCounter是否属于当前线程
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            //gc，防止内存泄露
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }
    //前面负责更新HoldCounter
    //后面负责更新state
    for (;;) {
        int c = getState();
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
```

```java
//尝试获取读锁
//以下条件才能获取读锁：
//1、写锁必须为0,或者当前线程持有写锁。这条件不符合时直接退出
//2、readerShouldBlock返回false。这条件不符合时还会尝试fullTryAcquireShared
//3、获取后读锁没有溢出。这条件不符合时还会尝试fullTryAcquireShared
//4、CAS成功。这条件不符合时还会尝试fullTryAcquireShared
protected final int tryAcquireShared(int unused) {
    Thread current = Thread.currentThread();
    int c = getState();
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;
    int r = sharedCount(c);
    if (!readerShouldBlock() &&
        r < MAX_COUNT &&
        compareAndSetState(c, c + SHARED_UNIT)) {
        if (r == 0) {
            //注意，firstReader和firstReaderHoldCount不是volatile的
        	//这里能保证这两个变量不会参与到多线程竞争中
            //在int c = getState();到CAS(c, c+SHARED_UNIT)这之间能保证c没有变化(始终准确)，那么r就没有变化
            //CAS成功相当于先把结果写上，然后这个if内的语句相当于在结果确定的情况下把过程给补上
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            firstReaderHoldCount++;
        } else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                //到达这一块，说明cachedHoldCounter属于当前线程
                //在tryReleaseShared中，如果释放后持有读锁的数量为0，那么会将线程的HoldCounter清空
                //因此存在一种情况，线程的本地变量为空，但cachedHoldCounter不会空
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    //完全版本的tryAcquire
    return fullTryAcquireShared(current);
}
```

```java
//完全版的tryAcquire
final int fullTryAcquireShared(Thread current) {
    HoldCounter rh = null;
    for (;;) {
        int c = getState();
        if (exclusiveCount(c) != 0) {
            if (getExclusiveOwnerThread() != current)
                return -1;
            // else we hold the exclusive lock; blocking here
            // would cause deadlock.
        } else if (readerShouldBlock()) {
            // Make sure we're not acquiring read lock reentrantly
            if (firstReader == current) {
                // assert firstReaderHoldCount > 0;
            } else {
                if (rh == null) {
                    rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current)) {
                        rh = readHolds.get();
                        if (rh.count == 0)
                            readHolds.remove();
                    }
                }
                if (rh.count == 0)
                    return -1;
            }
        }
        if (sharedCount(c) == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            if (sharedCount(c) == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                if (rh == null)
                    rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
                cachedHoldCounter = rh; // cache for release
            }
            return 1;
        }
    }
}
```