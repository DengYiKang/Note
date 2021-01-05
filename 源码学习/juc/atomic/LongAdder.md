# LongAdder

`LongAccumulator`与`DoubleXXX`跟`LongAdder`的设计与逻辑大同小异，这里只分析`LongAdder`。

`LongAdder`继承自`Striped64`类，实现了`Serializable`接口。

在高并发的情况下，`LongAdder`会更优于`AtomicLong`，详情见`Striped64`类。

更新操作：

```java
public void add(long x) {
    Cell[] as; long b, v; int m; Cell a;
    if ((as = cells) != null || !casBase(b = base, b + x)) {
        //Striped64的longAccumulate需要传入uncontended参数，当uncontended为false时(即cas失败)，
        //longAccumulate将rehash重试
        boolean uncontended = true;
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[getProbe() & m]) == null ||
            !(uncontended = a.cas(v = a.value, v + x)))
            longAccumulate(x, null, uncontended);
    }
}
```

求和与重置操作：

```java
public long sum() {
    Cell[] as = cells; Cell a;
    long sum = base;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}

public void reset() {
    Cell[] as = cells; Cell a;
    base = 0L;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                a.value = 0L;
        }
    }
}

public long sumThenReset() {
    Cell[] as = cells; Cell a;
    long sum = base;
    base = 0L;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null) {
                sum += a.value;
                a.value = 0L;
            }
        }
    }
    return sum;
}
```

注意它们都不是原子的！需要保证没有其他线程进行更新操作的情况下调用它们。

`LongAdder`中还定义了帮助序列化的嵌套类：

```java
private static class SerializationProxy implements Serializable {
    private static final long serialVersionUID = 7249069246863182397L;

    /**
     * The current value returned by sum().
     * @serial
     */
    private final long value;

    SerializationProxy(LongAdder a) {
        value = a.sum();
    }

    /**
     * Return a {@code LongAdder} object with initial state
     * held by this proxy.
     *
     * @return a {@code LongAdder} object with initial state
     * held by this proxy.
     */
    private Object readResolve() {
        LongAdder a = new LongAdder();
        a.base = value;
        return a;
    }
}
private Object writeReplace() {
	return new SerializationProxy(this);
}
private void readObject(java.io.ObjectInputStream s) 
    throws java.io.InvalidObjectException {
    throw new java.io.InvalidObjectException("Proxy required");
}
```

因为父类`Striped64`中的`base`与`cells`变量均为`transient`的，不能用于序列化（序列化后对应值为jvm初始值，即0、null、false等等），因此有必要在`LongAdder`中进行将sum值序列化。