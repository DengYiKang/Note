# AtomicInteger

atomic包下的AtomicXXX的实现大同小异，因此在这里只分析`AtomicInteger`类。

`AtomicInteger`继承自`Number`抽象类，实现了`java.io.Serializable`接口。

`Number`抽象类中主要定义了向各种基本类型转化的方法，是所有表数值的类的超类。

`AtomicInteger`定义了四个变量：

```java
private static final long serialVersionUID = 6214790243416807050L;

// setup to use Unsafe.compareAndSwapInt for updates
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;
private volatile int value;
```

`Unsafe`类是实现原子操作的基础，所有的方法都通过`Unsafe`类来实现。

`valueOffset`保存的是`value`变量的偏移地址，通过`Unsafe`的`objectFieldOffset`来实现的，在调用`Unsafe`的方法时需要传入`valueOffset`参数：

```java
static {
	try {
		valueOffset = unsafe.objectFieldOffset
		(AtomicInteger.class.getDeclaredField("value"));
	} catch (Exception ex) { throw new Error(ex); }
}
```

`AtomicInteger`的构造方法、get、set方法，由于`value`被`volatile`修饰，因此这些方法都具有`volatile`语义：

```java
public AtomicInteger(int initialValue) {
	value = initialValue;
}

public AtomicInteger() {
}

public final int get() {
	return value;
}

public final void set(int newValue) {
	value = newValue;
}
```

值得注意的是`lazySet`方法：

```java
/**
* Eventually sets to the given value.
*/
public final void lazySet(int newValue) {
	unsafe.putOrderedInt(this, valueOffset, newValue);
}
```

`lazySet`的语义是该写操作保证不会与之前的写操作重排，但可能与后续的操作重排，直到`volatile write`或同步动作的发生（`volatile write`或同步动作不允许其后的指令重排在自身前面）。即该写操作对于其他线程可能不立即可见。主要用于将结点的值置空，以方便垃圾回收，但是需要保证其他线程看到该结点不为空的一段时间内是无影响的（因为不立即可见，所以一段时间非空）。在这种情况，你可以避免`volatile write`或同步动作的置空操作带来的资源消耗，使你的代码性能更高效。

其他的原子方法，它们的语义都能从命名看出：

```java
public final int getAndSet(int newValue);
public final int getAndIncrement();
public final int getAndDecrement();
public final int getAndAdd(int delta);
public final int incrementAndGet();
public final int decrementAndGet();
public final int addAndGet(int delta);
```

特别的，我们需要关注`compareAndSet`方法：

```java
/**
* Atomically sets the value to the given updated value
* if the current value {@code ==} the expected value.
*
* @param expect the expected value
* @param update the new value
* @return {@code true} if successful. False return indicates that
* the actual value was not equal to the expected value.
*/
public final boolean compareAndSet(int expect, int update) {
	return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

`Unsafe`类的`compareAndSwapInt`俗称CAS操作，判断某个内存中的位置`valueOffset`的值是否为期待值`expect`，如果是那就把他替换为`update`。在一些情况下替代`synchronized`，避免过多的消耗。

可是有了`volatile`语义，为什么还需要CAS操作？因为虽然get和set方法是直接读取和写入内存的，但是get到的值肯定是需要拷贝到本地来做运算处理，但是在运算期间内存的值可能发生了变化，例如`i++`操作，如另一个线程对`i`进行了更新，而本线程仍保存的旧值，那么将覆盖另一个线程的结果。

值得注意的是还有`weakCompareAndSet`方法：

```java
/**
* May failspuriously and does not provide ordering guarantees, so is
* only rarely an appropriate alternative to {@code compareAndSet}.
*/
public final boolean weakCompareAndSet(int expect, int update) {
	return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

`weakCompareAndSet`提供弱化的CAS操作，但其调用代码与`CompareAndSet`相同，stackoverflow论坛上有人猜测：虽然代码一样，但可能会被JIT编译器重写以得到不同的机器码。

`AtomicInteger`类的一些方法用到了CAS操作：

```java
public final int getAndUpdate(IntUnaryOperator updateFunction) {
	int prev, next;
	do {
		prev = get();
		next = updateFunction.applyAsInt(prev);
	} while (!compareAndSet(prev, next));
    return prev;
}
public final int updateAndGet(IntUnaryOperator updateFunction) {
	int prev, next;
	do {
		prev = get();
		next = updateFunction.applyAsInt(prev);
	} while (!compareAndSet(prev, next));
	return next;
}
public final int getAndAccumulate(int x,IntBinaryOperator accumulatorFunction) {
	int prev, next;
	do {
		prev = get();
		next = accumulatorFunction.applyAsInt(prev, x);
	} while (!compareAndSet(prev, next));
	return prev;
}
public final int accumulateAndGet(int x,IntBinaryOperator accumulatorFunction) {
	int prev, next;
	do {
		prev = get();
		next = accumulatorFunction.applyAsInt(prev, x);
	} while (!compareAndSet(prev, next));
	return next;
}
```

要注意，CAS只能保证两个时刻的某个位置`valueOffset`的值是一致的，并不能保证这之间的时间段该位置的值没有发生变化。例如该位置保存这链表的头结点，期间发生了变动，虽然最终头结点还是原值，但链表可能已经发生了变化。这个问题也称为"ABA"问题。