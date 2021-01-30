# ReentrantReadWriteLock

[TOC]



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
//注意没有自旋，因为写锁只能由一个线程拥有，无需考虑竞争
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
//注意没有自旋，但需要CAS
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
//注意没有自旋，但需要CAS
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
//最后对state的CAS有自旋
//一般tryAcquire没有自旋，因为如果某个条件不具备，那么可能将一直不具备
//tryRelease一般有自旋，因为某个条件不具备时已经在HoldCounter处抛出异常，到达自旋处时所有的条件都已经具备
//且一直会是合法的
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
//这里与完全版的区别在于，完全版允许readerShouldBlock返回true且持有读锁的情况下重入读锁
//而该方法不允许重入（但最终调用fullTryAcquireShared是允许重入的）
//该方法的CAS没有自旋
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
            //同时更新cache
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
            //到达这里，表示持有写锁，允许获得读锁
        } else if (readerShouldBlock()) {
            //如果当前线程不是同步队列的head后继，当时它已经持有读锁，那么也是允许插队重入的
            if (firstReader == current) {
                // assert firstReaderHoldCount > 0;
            } else {
                if (rh == null) {
                    rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current)) {
                        rh = readHolds.get();
                        if (rh.count == 0)
                            //方便gc
                            readHolds.remove();
                    }
                }
                if (rh.count == 0)
                    //当前线程不是同步队列的head后继，且没有持有锁，不能插队
                    return -1;
            }
        }
        //到达这里表示可以获取读锁了
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
                //更新cache
                cachedHoldCounter = rh; 
            }
            return 1;
        }
    }
}
```

```java
//除了没有调用readerShouldBlock以及fullTryAcquireShared方法，其他与tryAcquireShared方法相同
final boolean tryReadLock() {
    Thread current = Thread.currentThread();
    for (;;) {
        int c = getState();
        if (exclusiveCount(c) != 0 &&
            getExclusiveOwnerThread() != current)
            return false;
        int r = sharedCount(c);
        if (r == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            if (r == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                HoldCounter rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    cachedHoldCounter = rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
            }
            return true;
        }
    }
}
```

## 锁降级和重入

一个线程持有写锁，可以继续获得读锁。一个线程同时拥有写锁和读锁，如果释放了写锁，那么就称写锁降级为了读锁。

+ 一个线程持有读锁，继续去获得读锁——锁的重入

+ 一个线程持有写锁，继续去获得读锁——锁的重入
+ 同时拥有读写锁，先释放写锁——锁降级

## 总结

+ `ReentrantReadWriteLock`内部定义了两个锁，分别是读锁和写锁
+ AQS子类sync定义了`HoldCounter`类来存储拥有读锁数量和对应线程的id，将它与`ThreadLocal`结合作为各线程的私有变量
+ AQS子类sync维护了`cachedHoldCounter`、`firstReader`和`firstReaderHoldCount`来作为缓存，提高速度
+ 读锁和写锁共用一个AQS队列，state的高16位为读锁的拥有量，state的低16位为写锁的拥有量
+ 一个线程具备获取写锁的条件：
	+ 读锁为0
	+ 写锁获取不超过上限
	+ 写锁为0或者当前线程为写锁的独占线程
	+ 公平模式下为head的后继，非公平无所谓
+ 一个线程具备获取读锁的条件：
	+ 当前写锁为0或者当前线程持有写锁
	+ 读锁获取不超过上限
	+ 如果当前线程持有读锁，那么在公平模式下即使不为head后继也允许获取
	+ 如果当前线程不持有读锁，且在非公平模式下不为head后继，那么只能排队等待