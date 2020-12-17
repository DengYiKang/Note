# AtomicIntegerArray

atomic包下的AtomicXXXArray的实现大同小异，因此在这里只分析`AtomicIntegerArray`类。

`AtomicIntegerArray`实现了`java.io.Serializable`接口。

有以下变量：

```java
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final int base = unsafe.arrayBaseOffset(int[].class);
private static final int shift;
private final int[] array;
```

其中`Unsafe`类就不需要解释了，`base`与`shift`变量用于辅助计算对应数组某个下标的内存地址。

**但是要注意，`array`变量并没有用`volatile`修饰！Java允许你用`volatile`对数组进行修饰，但是并不能赋予数组元素`volatile`语义，只能赋予引用本身`volatile`语义(例如在这里的`array`引用)。如果非要用`volatile`修饰数组，那么在赋值的时候应该替换该引用本身来实现`volatile`语义，比如这样赋值：`array=new_array`。那么在`AtomicIntegerArray`中如何实现`volatile`语义呢？其实是通过调用`Unsafe`类的`getIntVolatile`以及`putIntVolatile`来保证的。**

`checkedByteOffset`以及`byteOffset`方法用于辅助计算`array[i]`的内存地址：

```java
private long checkedByteOffset(int i) {
	if (i < 0 || i >= array.length)
		throw new IndexOutOfBoundsException("index " + i);
	return byteOffset(i);
}

private static long byteOffset(int i) {
	return ((long) i << shift) + base;
}
```

注意`array`变量没有用`volatile`修饰，因此需要调用`getIntVolatile`以及`putIntVolatile`来保证原子性。

```java
/**
* Gets the current value at position {@code i}.
*/
public final int get(int i) {
	return getRaw(checkedByteOffset(i));
}
private int getRaw(long offset) {
	return unsafe.getIntVolatile(array, offset);
}
/**
* Sets the element at position {@code i} to the given value.
*/
public final void set(int i, int newValue) {
	unsafe.putIntVolatile(array, checkedByteOffset(i), newValue);
}
```

注意`getIntVolatile`与`setIntVolatile`是`volatile`语义的，即`getIntVolatile`直接从内存里面取值，`setIntVolatile`直接将值写入内存，它们都被禁止使用本地缓存。

其他的方法的语义与实现与`AtomicXXX`类的类似：

```java
public final void lazySet(int i, int newValue) {
	unsafe.putOrderedInt(array, checkedByteOffset(i), newValue);
}
public final int getAndSet(int i, int newValue) {
	return unsafe.getAndSetInt(array, checkedByteOffset(i), newValue);
}
/*注意compareAndSet方法的实现与AtomicInteger类对应的方法实现完全相同
* 都是调用了Unsafe类的compareAndSwapInt方法
*/
public final boolean compareAndSet(int i, int expect, int update) {
	return compareAndSetRaw(checkedByteOffset(i), expect, update);
}
private boolean compareAndSetRaw(long offset, int expect, int update) {
	return unsafe.compareAndSwapInt(array, offset, expect, update);
}
public final boolean weakCompareAndSet(int i, int expect, int update) {
	return compareAndSet(i, expect, update);
}
public final int getAndIncrement(int i) {
	return getAndAdd(i, 1);
}
public final int getAndDecrement(int i) {
	return getAndAdd(i, -1);
}
public final int getAndAdd(int i, int delta) {
	return unsafe.getAndAddInt(array, checkedByteOffset(i), delta);
}
public final int incrementAndGet(int i) {
	return getAndAdd(i, 1) + 1;
}
public final int decrementAndGet(int i) {
	return getAndAdd(i, -1) - 1;
}
public final int addAndGet(int i, int delta) {
	return getAndAdd(i, delta) + delta;
}
public final int getAndUpdate(int i, IntUnaryOperator updateFunction) {
	long offset = checkedByteOffset(i);
	int prev, next;
	do {
		prev = getRaw(offset);
		next = updateFunction.applyAsInt(prev);
	} while (!compareAndSetRaw(offset, prev, next));
	return prev;
}
//实现跟上述方法类似，通过CAS来实现
public final int updateAndGet(int i, IntUnaryOperator updateFunction);
//同上
public final int getAndAccumulate(int i, int x, IntBinaryOperator accumulatorFunction);
//同上
public final int accumulateAndGet(int i, int x, IntBinaryOperator accumulatorFunction);

```

