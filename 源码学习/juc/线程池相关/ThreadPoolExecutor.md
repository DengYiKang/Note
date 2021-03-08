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
```

不难发现，这几个线程池状态无论后面的29位bit是什么，它们的大小顺序是完全的从小到大的顺序。

![在这里插入图片描述](../../../pic/41)