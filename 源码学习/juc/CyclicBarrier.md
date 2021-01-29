# CyclicBarrier

[TOC]



### 与CountDownLatch的区别

CountDownLatch只使用了同步队列，而CyclicBarrier既使用了同步队列也使用了条件队列。

CountDownLatch基于共享锁，而CyclicBarrier基于独占锁。

CountDownLatch一般用于两个场景：1、一个线程发令，n个线程同时开始；2、一个线程等待n个线程完全完成后才能开始运行。而CyclicBarrier强化了第二种场景，前n-1个线程到达屏障后等待，直到第n个线程到达屏障，当所有n个线程到达屏障时，可以执行用户传入的某项任务，然后对所有的线程进行signal，依次唤醒，后执行。

CountDownLatch只能使用一次；CyclicBarrier可以使用多次，每次为不同的generation。

CyclicBarrier遵循`all-or-none breakage model`的原则，同一代线程要么都正常在`CyclicBarrier#await`返回，要么都抛出异常(某个线程超时、中断将设置broken为true，其他线程会抛出broken异常)。

### 源码

#### 成员变量

```java
private static class Generation {
    boolean broken = false;
}

//可重入的独占锁，非公平
private final ReentrantLock lock = new ReentrantLock();
//lock对应的条件队列，到达屏障的线程会执行await进入条件队列，等待最后一个线程到来执行signalAll
private final Condition trip = lock.newCondition();
//表示每次通过屏障所需要的线程数量
private final int parties;
//当所有线程到达屏障时，执行barrierCommand，可以为null
private final Runnable barrierCommand;
//每一组通过屏障的线程叫做一代
private Generation generation = new Generation();
//当前还需要多少个线程到达屏障
private int count;
```

#### 构造器

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

public CyclicBarrier(int parties) {
    this(parties, null);
}
```

#### 辅助方法

```java
//signal所有等待在屏障上的线程，生成新的generation
//只能在持有锁的情况下调用
private void nextGeneration() {
    //signal所有await的线程
    trip.signalAll();
    //重新设置新的generation
    count = parties;
    generation = new Generation();
}
```

```java
//将broken设置为true,在await状态下的线程被唤醒后发现broken时，会抛出Broken异常
//只能在持有锁的情况下调用
private void breakBarrier() {
    generation.broken = true;
    count = parties;
    trip.signalAll();
}
```

```java
//一般在线程抛出broken异常时调用
//重置屏障到初始状态，如果一些线程仍在await状态,察觉到屏障是broken的，那么它们将抛出BrokenBarrierException
//reset需要获取锁，否则，signal后，线程还未到判断是否broken的那一步而nextGeneration执行了，
//那么那些旧generation的线程将不会抛出broken异常，而是会继续执行
public void reset() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        breakBarrier();   // break the current generation
        nextGeneration(); // start a new generation
    } finally {
        lock.unlock();
    }
}
```

#### wait

```java
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}
```
```java
//超时机制的await
public int await(long timeout, TimeUnit unit)
    throws InterruptedException,
           BrokenBarrierException,
           TimeoutException {
    return dowait(true, unit.toNanos(timeout));
}
```

```java
/**
* timed表示是否有超时机制，当等待时长超过nanos时将调用breakBarrier
*/
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException, TimeoutException {
    //获取锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        //将当前generation保存下来，以便后续比较generation是否发生改变
        final Generation g = generation;

        //如果屏障broken了，那么抛出Broken异常
        //注意，这部分的判断自旋前和自旋后分别有一个
        if (g.broken)
            throw new BrokenBarrierException();

        //如果被中断了，那么调用breakBarrier，排除中断异常（其他await的线程抛出Broken异常）
        if (Thread.interrupted()) {
            breakBarrier();
            throw new InterruptedException();
        }

        //index表示还需要多少个线程到达屏障，也可以作为当前线程到达的编号
        int index = --count;
        if (index == 0) {
            // tripped，所有线程都到了，即当前线程最后一个到达屏障，最后一个线程执行barrierCommand
            //判断barrierCommand是否运行成功(为null也代表成功)，不成功则breakBarrier
            //成功则生成下一代，调用nextGeneration
            boolean ranAction = false;
            try {
                final Runnable command = barrierCommand;
                if (command != null)
                    command.run();
                ranAction = true;
                nextGeneration();
                return 0;
            } finally {
                if (!ranAction)
                    breakBarrier();
            }
        }

        //执行到这，说明当前线程不是最后一个线程
        //自旋，直到其他所有线程都到达了屏障或者屏障broken了或者被中断或者超时
        //（一直不知道这个自旋有什么用）
        for (;;) {
            try {
                if (!timed)
                    trip.await();
                else if (nanos > 0L)
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                //如果当前代是最新一代且当前代不是broken的，那么breakBarrier
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    //1、g！=generation，表示当前代不是最新代，不能breakBarrier，否则将影响最新代
                    //2、g==generation&&g.broken，当前代是最新代，且是broken，不需要再breakBarrier
                    //设置中断信息
                    Thread.currentThread().interrupt();
                }
            }

            //以下情况会抛出Broken异常
            //1、其他线程被中断，breakBarrier
            //2、最后一个线程执行barrierCommand出错，breakBarrier
            //3、拥有超时机制的某个线程超时，breakBarrier
            //4、reset方法执行，任意线程都可以调用reset
            if (g.broken)
                throw new BrokenBarrierException();

            //到达这里后，如果g！=generation，表示最后一个线程到达屏障且执行barrierCommand成功
            if (g != generation)
                return index;

            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        lock.unlock();
    }
}
```

### 一代线程通过屏障的完整流程

1、前n-1个线程到达屏障，将count减一，然后调用`Condition#await`阻塞。此时前n-1个线程处于条件队列上。

2、第n个线程到达屏障，将count减到0，然后执行barrierCommand。

#### 如果barrierCommand成功

3、调用nextGeneration，nextGeneration调用signalAll，前n-1个线程入队同步队列。接着第n个线程return，finally块中释放锁。

4、前n-1个线程依次获取锁，释放锁（注意ReentrantLock是非公平的）。

#### 如果barrierCommand出错

3、第n个线程调用breakBarrier，breakBarrier设置broken为true，signalAll，将前n-1个线程入队同步队列，执行finall的unlock释放锁。

4、前n-1个线程依次获取锁，发现broken为true，抛出broken异常，释放锁。

以上可以看出，n个线程要么都做要么都不做。

### 前n-1个线程发生中断或超时的流程

1、某个线程发生中断，那么抛出中断异常，执行breakBarrier，将所有在条件队列的线程入队同步队列（某些会直接唤醒去acquireQueued），释放锁；若发生超时，`nanos = trip.awaitNanos(nanos)`为负数，在后面的判断如果小于等于0那么也会执行breakBarrier，抛出超时异常。

2、在同步队列中竞争锁成功的线程，在`Condition#await`后的判断语句中抛出broken异常，释放锁；在新来的线程在`Condition#await`前的判断语句中抛出broken异常。

#### 第n个线程执行nextGeneration之前，前面的线程发生中断或超时

因为第n个线程执行`CyclicBarrier#await`时全程持有锁未释放锁，因此前面的线程将阻塞在`acquireQueued`里，所以这个问题可以转换成下一个问题。

#### 第n个线程执行nextGeneration之后，前面的线程发生中断或超时

nextGeneration后，前面的线程一定有`g!=generation`，即当前代不是最新代，就不会执行breakBarrier，当前代的broken也为false，因此会在`if (g != generation) return index;`正常退出。

超时同理。所以为什么下面的两个判断是这样的顺序：

```java
if (g != generation)
	return index;

if (timed && nanos <= 0L) {
	breakBarrier();
	throw new TimeoutException();
}
```
综上，如果最后一个线程顺利地通过屏障了，那么无论前面的线程是否发生中断或超时，它们都会顺利返回。

#### 第n+1个线程获得到的Generation局部变量，会不会是第一代的？

如果第n个线程的nextGeneration执行成功，那么第n+1个线程获得的是第二代的。

如果第n个线程的nextGeneration执行出错，那么第n+1个线程获得的是第一代的，不过马上会抛出broken异常。

#### 条件队列上的node的线程，肯定是同一代的吗？

是的。换代只有nextGeneration能换，并且还会调用signalAll，将所有在条件队列上的线程入队同步队列。

#### 同步队列上的node的线程，肯定是同一代的吗？

不一定。换代只有nextGeneration能换。第n个线程执行了nextGeneration后，前n-1个线程入队同步队列，它们是第一代。现在第二代的n个线程到来，因为非公平方式，它们都有可能抢到锁，如果它们都成功地抢到了锁，那么第2n个线程执行nextGeneration后，第n+1～2n-1个线程会入队同步队列。这时同步队列中存在不同代的node。

