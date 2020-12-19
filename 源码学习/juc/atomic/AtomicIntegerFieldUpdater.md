# AtomicIntegerFieldUpdater

`AtomicIntegerFielderUpdater`类基于反射，能够提供对指定的类的指定的`volatile int`字段的原子更新操作。

```java
//构造方法，传入指定的class以及字段，
public static <U> AtomicIntegerFieldUpdater<U> newUpdater(Class<U> tclass, 
                                                          String fieldName);
```

以下是提供的原子操作方法：

```java
public abstract boolean compareAndSet(T obj, int expect, int update);
public abstract boolean weakCompareAndSet(T obj, int expect, int update);
public abstract void set(T obj, int newValue);
public abstract void lazySet(T obj, int newValue);
public abstract int get(T obj);
public int getAndSet(T obj, int newValue);
public int getAndIncrement(T obj);
public int getAndDecrement(T obj);
public int getAndAdd(T obj, int delta);
public int incrementAndGet(T obj);
public int decrementAndGet(T obj);
public int addAndGet(T obj, int delta);
public final int getAndUpdate(T obj, IntUnaryOperator updateFunction);
public final int updateAndGet(T obj, IntUnaryOperator updateFunction);
public final int getAndAccumulate(T obj, int x,
                                      IntBinaryOperator accumulatorFunction);
public final int accumulateAndGet(T obj, int x,
                                      IntBinaryOperator accumulatorFunction);

```

使用`AtomicInteger`类还是`AtomicIntegerFieldUpdater`类？

使用`AtomicXXXFieldUpdater`类有以下好处：

+ 可以避免引入`AtomicXXX`类。
+ 当需要构造出大量的需要保证线程安全的对象时，可以仅仅将对应的域设置为`volatile`，用`AtomicXXXFieldUpdater`来统一操作，而不是将`AtomicXXX`嵌入其中。