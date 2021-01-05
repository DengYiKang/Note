# Striped64

当大量的线程需要访问一个`Atomic`类时，虽然能满足原子语义，但是吞吐量会大大减少。

对于一些更新操作，如增加某个数值或是减少某个数值，可以考虑将它们分摊到多个`Cell`中，每个`Cell`均实现了`cas`方法。最后需要总和时，只需要将它们累加返回。这样便能提高性能。

`Cell`类定义如下：

```java
@sun.misc.Contended 
static final class Cell {
    volatile long value;
    Cell(long x) { value = x; }
    final boolean cas(long cmp, long val) {
        return UNSAFE.compareAndSwapLong(this, valueOffset, cmp, val);
    }

    // Unsafe mechanics
    private static final sun.misc.Unsafe UNSAFE;
    private static final long valueOffset;
    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> ak = Cell.class;
            valueOffset = UNSAFE.objectFieldOffset
                (ak.getDeclaredField("value"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```

可以看出，`Cell`只是维护了一个`volatile`变量以及它的`cas`方法。注意`Cell`使用`@Contented`注解，因为在`Striped64`里维护了`Cell`数组，其名为`cells`，需要加入padding来解决FalseSharing的问题。

`Striped64`类对`cells`使用了懒初始化的方法，即如果不需要时，不对`cells`进行初始化。`cells`的长度为2的次方数。各个线程使用它们的hash值对`cells`进行访问，如果失败，将进行再hash。

注意`Striped64`类还定义了一个`base`的`volatile`类型变量。一般地，在没有竞争的情况下访问或是作为特殊情况的解决方案，这个特殊情况在后续会讲到。

在执行初始化，扩展，竞争等操作时，使用自旋锁的机制来实现，其锁为`cellsBusy`的`volatile`变量：

```java
//Spinlock (locked via CAS) used when resizing and/or creating Cells.
transient volatile int cellsBusy;
```

`longAccumulate`和`doubleAccumulate`方法是`Striped64`的核心方法，所有逻辑都在这里体现，因为两者的逻辑类似，因此这里只分析`longAccumulate`方法：

```java
final void longAccumulate(long x, LongBinaryOperator fn,
                          boolean wasUncontended) {
    int h;
    //h是当前线程的hash值
    if ((h = getProbe()) == 0) {
        ThreadLocalRandom.current();
        h = getProbe();
        wasUncontended = true;
    }
    boolean collide = false;                // True if last slot nonempty
    //自旋
    for (;;) {
        Cell[] as; Cell a; int n; long v;
        //如果cells已经初始化了且cells长度大于0
        if ((as = cells) != null && (n = as.length) > 0) {
            //因为是懒初始化的方式，因此探测的位置可能还未初始化
            if ((a = as[(n - 1) & h]) == null) {
                //如果还未初始化，那么初始化并赋值
                if (cellsBusy == 0) {
                    Cell r = new Cell(x);
                    //获取锁
                    if (cellsBusy == 0 && casCellsBusy()) {
                        boolean created = false;
                        try {               // Recheck under lock
                            Cell[] rs; int m, j;
                            if ((rs = cells) != null &&
                                (m = rs.length) > 0 &&
                                rs[j = (m - 1) & h] == null) {
                                rs[j] = r;
                                created = true;
                            }
                        } finally {
                            //释放锁
                            cellsBusy = 0;
                        }
                        //如果成功创建则退出自旋
                        if (created)
                            break;
                        //失败则自旋，继续尝试
                        continue;
                    }
                }
                collide = false;
            }
            //wasUncontended是外部传来的，如果上次的cas失败了，那么进行rehash操作
            //注意有break则表示退出方法，continue表示重试，否则进行rehash操作（后面有rehash代码块）
            else if (!wasUncontended)       // CAS already known to fail
                wasUncontended = true;      // Continue after rehash
            //探测的位置进行cas更新操作，成功则退出自旋
            else if (a.cas(v = a.value, ((fn == null) ? v + x :
                                         fn.applyAsLong(v, x))))
                break;
            //检查cells的大小是否超出cpu数，如果超出则继续自旋，避免进行后续的扩展操作
            else if (n >= NCPU || cells != as)
                collide = false;            // At max size or stale
            else if (!collide)
                collide = true;
            //扩展cells
            else if (cellsBusy == 0 && casCellsBusy()) {
                try {
                    if (cells == as) {
                        Cell[] rs = new Cell[n << 1];
                        for (int i = 0; i < n; ++i)
                            rs[i] = as[i];
                        cells = rs;
                    }
                } finally {
                    cellsBusy = 0;
                }
                collide = false;
                //重试
                continue;
            }
            //再hash
            h = advanceProbe(h);
        }
        //cells的长度为0，或为空，那么初始化成长度为2的Cell数组，并赋值
        else if (cellsBusy == 0 && cells == as && casCellsBusy()) {
            boolean init = false;
            try {                           // Initialize table
                if (cells == as) {
                    Cell[] rs = new Cell[2];
                    rs[h & 1] = new Cell(x);
                    cells = rs;
                    init = true;
                }
            } finally {
                cellsBusy = 0;
            }
            if (init)
                break;
        }
        //以上情况都失败了，直接对base进行cas更新
        else if (casBase(v = base, ((fn == null) ? v + x :
                                    fn.applyAsLong(v, x))))
            break;                          // Fall back on using base
    }
}
```

