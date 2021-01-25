# ReentrantLock

重入锁是一个独占锁，可重入指的是当某个线程持有锁时，它可以acquire多次。即当state=0时，没有线程持有锁，当不等于0时，某个线程持有锁，且这个线程acquire了state次。

源码比较简单：

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = 7373984872572414699L;
    private final Sync sync;

    
    //父类是非公平的
    abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -5179523762034025860L;

        abstract void lock();

        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }

        protected final boolean isHeldExclusively() {
            return getExclusiveOwnerThread() == Thread.currentThread();
        }

        final ConditionObject newCondition() {
            return new ConditionObject();
        }


        final Thread getOwner() {
            return getState() == 0 ? null : getExclusiveOwnerThread();
        }

        final int getHoldCount() {
            return isHeldExclusively() ? getState() : 0;
        }

        final boolean isLocked() {
            return getState() != 0;
        }

        /**
         * Reconstitutes the instance from a stream (that is, deserializes it).
         */
        private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
            s.defaultReadObject();
            setState(0); // reset to unlocked state
        }
    }

    /**
     * 非公平锁
     */
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;


        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }

    /**
     * 公平锁
     */
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }


        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }

	//默认是非公平的
    public ReentrantLock() {
        sync = new NonfairSync();
    }


    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }

    /**
     * 尝试获取锁
     * 成功返回true，失败返回false
     */
    public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }

    /**
     * 释放锁
     */
    public void unlock() {
        sync.release(1);
    }

    /**
     * 返回Condition实例，以使用对应的条件队列
     *
     */
    public Condition newCondition() {
        return sync.newCondition();
    }

    /**
     * 返回当前线程持有锁的数量，如果当前线程不持有锁，则返回0
     * 一般用于调试和测试
     */
    public int getHoldCount() {
        return sync.getHoldCount();
    }

    /**
     * 判断锁是否被当前线程持有
     * 跟Thread#holdsLock(Object)很像
     * 该方法一般用于调试和测试
     */
    public boolean isHeldByCurrentThread() {
        return sync.isHeldExclusively();
    }


    //是否被线程持有锁
    public boolean isLocked() {
        return sync.isLocked();
    }

    
    public final boolean isFair() {
        return sync instanceof FairSync;
    }
    
    //返回持有锁的线程，如果无则返回null
    protected Thread getOwner() {
        return sync.getOwner();
    }

    //判断同步队列上是否有线程等待
    public final boolean hasQueuedThreads() {
        return sync.hasQueuedThreads();
    }

    //判断thread是否在同步队列上
    public final boolean hasQueuedThread(Thread thread) {
        return sync.isQueued(thread);
    }

    //返回同步队列的长度
    public final int getQueueLength() {
        return sync.getQueueLength();
    }

    //返回同步队列上的所有线程
    protected Collection<Thread> getQueuedThreads() {
        return sync.getQueuedThreads();
    }

	//判断是否有线程在condition对应的条件队列上等待
    public boolean hasWaiters(Condition condition) {
        if (condition == null)
            throw new NullPointerException();
        if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
            throw new IllegalArgumentException("not owner");
        return sync.hasWaiters((AbstractQueuedSynchronizer.ConditionObject)condition);
    }

	//返回condition对应的条件队列上的等待线程数
    public int getWaitQueueLength(Condition condition) {
        if (condition == null)
            throw new NullPointerException();
        if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
            throw new IllegalArgumentException("not owner");
        return sync.getWaitQueueLength((AbstractQueuedSynchronizer.ConditionObject)condition);
    }

	//返回condition对应的条件队列上的等待线程
    protected Collection<Thread> getWaitingThreads(Condition condition) {
        if (condition == null)
            throw new NullPointerException();
        if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
            throw new IllegalArgumentException("not owner");
        return sync.getWaitingThreads((AbstractQueuedSynchronizer.ConditionObject)condition);
    }

    public String toString() {
        Thread o = sync.getOwner();
        return super.toString() + ((o == null) ?
                                   "[Unlocked]" :
                                   "[Locked by thread " + o.getName() + "]");
    }
}
```