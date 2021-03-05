# PriorityBlockingQueue

### 前言

`PriorityBlockingQueue`是一个无界阻塞队列，它的出队方式不再是FIFO，而是优先级高的先出队。其内部实现是最小堆，即堆顶元素是逻辑上最小的那个元素，也是最先出队的那个元素。简单的说，如果`a.compareTo(b) < 0`的话，那么`a`将先出队。

### 成员

```java
//默认数组的容量
private static final int DEFAULT_INITIAL_CAPACITY = 11;
//数组的最大容量，减8是因为有的虚拟机实现里数组的前8个字节用来存储别的东西
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
//内部数组，逻辑上是一个堆
private transient Object[] queue;
 //元素个数，小于等于queue.length
private transient int size;
//比较器
private transient Comparator<? super E> comparator;
//唯一的锁
private final ReentrantLock lock;
//队列空时，出队线程将阻塞在这里
private final Condition notEmpty;
//相当于AQS的state，持有这个state才可以准备新数组以扩容
private transient volatile int allocationSpinLock;
//用作序列化，只有在序列化/反序列化的时候是非空的
private PriorityQueue<E> q;
```
### 构造器
```java
public PriorityBlockingQueue() {
    this(DEFAULT_INITIAL_CAPACITY, null);
}

public PriorityBlockingQueue(int initialCapacity) {
    this(initialCapacity, null);
}

public PriorityBlockingQueue(int initialCapacity,
                             Comparator<? super E> comparator) {
    if (initialCapacity < 1)
        throw new IllegalArgumentException();
    this.lock = new ReentrantLock();
    this.notEmpty = lock.newCondition();
    this.comparator = comparator;
    this.queue = new Object[initialCapacity];
}

public PriorityBlockingQueue(Collection<? extends E> c) {
    this.lock = new ReentrantLock();
    this.notEmpty = lock.newCondition();
    boolean heapify = true; // true 表示需要进行堆排序
    boolean screen = true;  // true 表示需要扫描以检测有没有null value，如果有就抛出异常
    if (c instanceof SortedSet<?>) {
        SortedSet<? extends E> ss = (SortedSet<? extends E>) c;
        this.comparator = (Comparator<? super E>) ss.comparator();
        heapify = false;
    }
    else if (c instanceof PriorityBlockingQueue<?>) {
        PriorityBlockingQueue<? extends E> pq =
            (PriorityBlockingQueue<? extends E>) c;
        this.comparator = (Comparator<? super E>) pq.comparator();
        screen = false;
        if (pq.getClass() == PriorityBlockingQueue.class) // exact match
            heapify = false;//原本就是堆结构，不需要进行堆排序了
    }
    Object[] a = c.toArray();
    int n = a.length;
    // If c.toArray incorrectly doesn't return Object[], copy it.
    if (a.getClass() != Object[].class)
        a = Arrays.copyOf(a, n, Object[].class);
    if (screen && (n == 1 || this.comparator != null)) {
        for (int i = 0; i < n; ++i)
            if (a[i] == null)
                throw new NullPointerException();
    }
    this.queue = a;
    this.size = n;
    if (heapify)
        heapify();
}
```

### heapify

建堆，堆维护算法就不解释了。

```java
private void heapify() {
    Object[] array = queue;
    int n = size;
    int half = (n >>> 1) - 1;
    Comparator<? super E> cmp = comparator;
    if (cmp == null) {
        for (int i = half; i >= 0; i--)
            siftDownComparable(i, (E) array[i], array, n);
    }
    else {
        for (int i = half; i >= 0; i--)
            siftDownUsingComparator(i, (E) array[i], array, n, cmp);
    }
}
```

### 入队

由于`PriorityBlockingQueue`是无界的，所以入队是不可能因为队列满而阻塞的，但有可能因为内存耗尽而抛出`OutOfMemoryError`。

#### offer

`add`直接调用`offer`，这里就直接分析`offer`。

```java
public boolean add(E e) {
    return offer(e);
}
```

```java
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    final ReentrantLock lock = this.lock;
    lock.lock();
    int n, cap;
    Object[] array;
    while ((n = size) >= (cap = (array = queue).length))//满了，需要扩容
        tryGrow(array, cap);
    try {
        Comparator<? super E> cmp = comparator;
        if (cmp == null)
            siftUpComparable(n, e, array);
        else
            siftUpUsingComparator(n, e, array, cmp);
        size = n + 1;
        notEmpty.signal();
    } finally {
        lock.unlock();
    }
    return true;
}
```

### 扩容

```java
//调用这个方法时已经获取了锁，必须要保证这个方法退出时，锁的状态不变
//因此开头释放锁后，退出前必须先获得锁
private void tryGrow(Object[] array, int oldCap) {
    lock.unlock(); // 先释放锁，新建数组的同时，允许别的线程对旧table进行并发操作
    Object[] newArray = null;
    if (allocationSpinLock == 0 &&
        UNSAFE.compareAndSwapInt(this, allocationSpinLockOffset,
                                 0, 1)) {//同时只能有一个线程进行扩容操作
        try {
            //新容量的计算公式：
			//1、如果oldCap < 64, 新容量等于2(oldCap + 1)
			//2、如果oldCap >=64, 新容量等于1.5oldCap
            //3、经过前面的步骤算出的容量如果大于MAX_ARRAY_SIZE，那就oldCap+1，如果还溢出，那就抛出异常
            int newCap = oldCap + ((oldCap < 64) ?
                                   (oldCap + 2) : // grow faster if small
                                   (oldCap >> 1));
            if (newCap - MAX_ARRAY_SIZE > 0) {    //可能会溢出
                int minCap = oldCap + 1;
                if (minCap < 0 || minCap > MAX_ARRAY_SIZE)//已经溢出了
                    throw new OutOfMemoryError();
                newCap = MAX_ARRAY_SIZE;//赋值允许的最大长度
            }
            if (newCap > oldCap && queue == array)//期间数组的引用没有发生变化
                newArray = new Object[newCap];
        } finally {
            allocationSpinLock = 0;//释放锁
        }
    }
    if (newArray == null) // queue == array这个条件不成立，其他线程正在扩容
        Thread.yield();
    lock.lock();//获取全局锁
    if (newArray != null && queue == array) {
        queue = newArray;
        System.arraycopy(array, 0, newArray, 0, oldCap);
    }
}
```

为什么需要`queue==array`？因为`allocationSpinLock`与`lock`是分隔开的。假设线程1新数组开辟空间完毕，释放`allocationSpinLock`，获取`lock`，将新数组的引用写入`queue`，而此时线程2获取`allocationSpinLock`成功，开辟空间，发现`queue`的引用发生了变化，就不会执行`newArray = new Object[newCap];`，因此`newArray`为null，后面将不会进行扩容的最后阶段（更新`queue`）。

### 出队

因为队列可能没有元素，所以出队线程是可能阻塞在AQS条件队列里的。

#### poll

```java
//只尝试一次
public E poll() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return dequeue();
    } finally {
        lock.unlock();
    }
}
```

### peek

```java
public E peek() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return (size == 0) ? null : (E) queue[0];
    } finally {
        lock.unlock();
    }
}
```

### 内部删除

```java
public boolean remove(Object o) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        int i = indexOf(o);
        if (i == -1)
            return false;
        removeAt(i);
        return true;
    } finally {
        lock.unlock();
    }
}
```

```java
private void removeAt(int i) {
    Object[] array = queue;
    int n = size - 1;
    if (n == i) // removed last element
        array[i] = null;
    else {
        E moved = (E) array[n];
        array[n] = null;
        Comparator<? super E> cmp = comparator;
        if (cmp == null)
            siftDownComparable(i, moved, array, n);
        else
            siftDownUsingComparator(i, moved, array, n, cmp);
        if (array[i] == moved) {//说明该位置没有下沉
            if (cmp == null)
                siftUpComparable(i, moved, array);
            else
                siftUpUsingComparator(i, moved, array, cmp);
        }
    }
    size = n;
}
```

### 迭代器

`PriorityBlockingQueue`的迭代器也是弱一致性的，而且它弱得都有点离谱，因为在迭代器对象初始化的时候，就复制了一个新数组出来。也就是说，从初始化的时间节点之后，元素被从`PriorityBlockingQueue`中删除了迭代器也不管，新元素加入了`PriorityBlockingQueue`迭代器也不管。

### 总结

+ 使用最小堆来实现优先级出队，但内部成员是数组，只是逻辑是个堆。
+ 两个元素a、b，如果a的优先级更高，那么肯定a.compareTo(b) < 0。
+ 对于`PriorityBlockingQueue`来说，只需要时刻知道队列哪个元素最小，其他元素的顺序并不重要。所以使用堆这种数据结构再合适不过。
+ 元素比较方式有两种，`Comparable`和`Comparator`。但元素不一定支持`Comparable`，所以`PriorityBlockingQueue`的声明不能写成`PriorityBlockingQueue<E extends Comparable<E>>`这样的泛型自限定。
+ 堆的内部操作只有两种：冒泡上移和冒泡下沉。
+ `PriorityBlockingQueue`不允许存储null元素。
+ 迭代器初始化时会保留一份当前数组的拷贝，所有操作都在拷贝的数组上，但是`remove`还是有效的，它会在原数组上删除。