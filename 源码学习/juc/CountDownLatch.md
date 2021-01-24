# CountDownLatch

CountDownLatch比较简单，这里直接贴源码：

```java
//共享锁,一次性，state减到0就不能用了
//一开始看到await方法以为是调用了AQS的条件队列的相关方法，但其实只调用了同步队列的相关方法
//需要阻塞的对象调用await
//起触发器的作用的对象调用release
public class CountDownLatch {
    //AQS的子类
    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;

        Sync(int count) {
            setState(count);
        }

        int getCount() {
            return getState();
        }

        //注意跟常理相反
        //在AQS中state一般表示资源量，但在这里state=0才能获取锁成功，0是触发器
        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }

        //释放操作，跟常理相反
        //在AQS中释放操作会使state增加，而这里会使state减少
        protected boolean tryReleaseShared(int releases) {
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
    }

    private final Sync sync;

    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }

	//需要等待时调用，当state=0时退出await
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }

	//超时机制的await
    //要注意的是，超时await将不再等待，返回false
    public boolean await(long timeout, TimeUnit unit)
        throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }

	//将锁的数量减少1
    public void countDown() {
        sync.releaseShared(1);
    }

    //获取当前state
    public long getCount() {
        return sync.getCount();
    }

    public String toString() {
        return super.toString() + "[Count = " + sync.getCount() + "]";
    }
}
```

```java
//使用例子
//startSignal用来作为触发器，当调用countDown时，N个线程才能运行
//doneSignal用来等待所有事务的完成，当所有事务countDown后，某个任务才能运行
class Driver {
   void main() throws InterruptedException {
     CountDownLatch startSignal = new CountDownLatch(1);
     CountDownLatch doneSignal = new CountDownLatch(N);

     for (int i = 0; i < N; ++i)
       new Thread(new Worker(startSignal, doneSignal)).start();

     doSomethingElse();            // don't let run yet
     startSignal.countDown();      // let all threads proceed
     doSomethingElse();
     doneSignal.await();           // wait for all to finish
   }
 }

 class Worker implements Runnable {
   private final CountDownLatch startSignal;
   private final CountDownLatch doneSignal;
   Worker(CountDownLatch startSignal, CountDownLatch doneSignal) {
     this.startSignal = startSignal;
     this.doneSignal = doneSignal;
   }
   public void run() {
     try {
       startSignal.await();
       doWork();
       doneSignal.countDown();
     } catch (InterruptedException ex) {} // return;
   }
```