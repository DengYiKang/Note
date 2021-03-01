# ConcurrentSkipListMap

### 简介

是接口`ConcurrentNavigableMap`的实现。这类map是有序的，由key的自然顺序（即key类的CompareTo方法）或者构造方法中传入的Comparator决定的。

内部数据结构是跳表的并发变式。`containsKey`、`get`、`put`、`remove`的平均时间复杂度为$O(logn)$。Insertion、removal、update以及各种访问操作都可以被多线程并发、安全地执行。

Iterators和spliterators是弱一致性的。

对key升序的iterator要比降序的快。

注意，`size`方法并不是常数时间的。因为并发，`size`方法返回的结果可能并不准确。并且批量操作`putAll`、`equals`、`toArray`、`containsValue`、`clear`不能保证原子性，例如在iterator的遍历过程中，如果执行了`putAll`操作，iterator可能只能观察到部分新添加的元素。

注意，不允许null的key或者value，因为返回值为null不好与空区别开（应该可以定义代表NULL的Object来解决？）。

