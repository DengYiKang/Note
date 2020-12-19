# AtomicMarkableReference

`AtomicMarkableReference<T>`和`AtomicStampedReference<T>`非常类似。

前者维护了一个二元组`[reference, boolean]`，后者维护了一个二元组`[reference, int]`，都提供了对应的原子操作。

二元组是以`Pair`类的形式存储的（自定义`Pair`类）：

```java
private static class Pair<T> {
	final T reference;
	final boolean mark;
	private Pair(T reference, boolean mark) {
		this.reference = reference;
		this.mark = mark;
	}
	static <T> Pair<T> of(T reference, boolean mark) {
		return new Pair<T>(reference, mark);
	}
}

private volatile Pair<V> pair;
```

大部分与`AtomicInteger`类相同。

值得注意的是`compareAndSet`方法和`attemptMark`方法：

```java
public boolean compareAndSet(V expectedReference, V newReference, 
                             boolean expectedMark, boolean newMark) {
	Pair<V> current = pair;
	return
		expectedReference == current.reference &&
		expectedMark == current.mark &&
		((newReference == current.reference &&
		newMark == current.mark) ||
		casPair(current, Pair.of(newReference, newMark)));
}
public boolean attemptMark(V expectedReference, boolean newMark) {
	Pair<V> current = pair;
	return
		expectedReference == current.reference &&
		(newMark == current.mark ||
		casPair(current, Pair.of(expectedReference, newMark)));
}
```

它们先把`pair`赋值于本地变量，将一些操作放在本地变量赋值与CAS操作间，保证它们的有效性。即使失效，最后`casPair`方法也会失败，并不会导致任何意外的结果。

