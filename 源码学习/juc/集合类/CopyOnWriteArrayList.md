# CopyOnWriteArrayList

[TOC]

### 简介

线程安全的ArrayList。

### 成员变量

```java
//任何修改性质的方法都需要获取锁
final transient ReentrantLock lock = new ReentrantLock();

//内部数据结构
private transient volatile Object[] array;
```

### 基本方法

任何非修改性质的方法都不会获取锁，保留了读操作的高性能。

任何修改性质的方法都需要获取锁，高内存消耗。

> 因此可能发生获取的数据可能是旧数据。

任何对数据的修改，都会对内部数据结构`array`赋予新值，因此叫做CopyOnWrite，这样避免了因为并发修改iterator抛出`ConcurrentModificationException`异常。

大部分的修改方法只是简单的获取锁后进行处理，这里只分析一些有趣的代码：

```java
//将remove功能分成两个函数，将锁的粒度精细化，降低消耗，提高吞吐量
//先获取要删除对象的索引，这个索引可能是旧索引，这期间数据可能会发生变化
public boolean remove(Object o) {
    Object[] snapshot = getArray();
    int index = indexOf(o, snapshot, 0, snapshot.length);
    return (index < 0) ? false : remove(o, snapshot, index);
}

//snapshot与index都可能是旧数据
private boolean remove(Object o, Object[] snapshot, int index) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] current = getArray();
        int len = current.length;
        //当snapshot != current时，数据在这期间被修改了
        if (snapshot != current) findIndex: {
            //保证遍历0~prefix不越界
            int prefix = Math.min(index, len);
            for (int i = 0; i < prefix; i++) {
                //这里很巧妙
                //在旧的数组里，要删除的对象在第index位置，即snapshot[0],...,snapshot[index-1]都不是
                //即snapshot[0],...,snapshot[prefix-1]都不是要删除的对象
                //这里直接写if(eq(o, current[i]))是没问题的，这里的目的是提高性能
                //调用eq的消耗远比直接判断引用更大
                //因此如果current[i] == snapshot[i]，那么current[i]肯定不是要删除的对象，因此直接短路后面的判断
                if (current[i] != snapshot[i] && eq(o, current[i])) {
                    index = i;
                    break findIndex;
                }
            }
            //如果index>=len，那么prefix=len，意味着之前已经遍历完所有的元素了，肯定无了
            if (index >= len)
                return false;
            //下面都是index<len的情况
            if (current[index] == o)
                break findIndex;
            index = indexOf(o, current, index, len);
            if (index < 0)
                return false;
        }
        Object[] newElements = new Object[len - 1];
        System.arraycopy(current, 0, newElements, 0, index);
        System.arraycopy(current, index + 1,
                         newElements, index,
                         len - index - 1);
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

> `if (current[i] != snapshot[i] && eq(o, current[i]))`这句代码很巧妙，具体看代码注释。

### COWIterator

注意iterator不提供对内部数据的任何修改方法，那么意味着iterator将不会抛出`ConcurrentModificationException`。

> 因为任何修改都会对`element`赋予新值，因此即使iterator已经构造出来了，其他地方对内部数据进行修改，iterator所引用的数组不会发生任何改变。因此也不需要维护ModCount变量。

### COWSubList

`COWSubList`提供了add、remove等方法，因此需要检查数组是否被修改。与ModCount原理一样，保存创建SubList时的数组引用`expectedArray`以及COW本体`l`，在被调用修改性质的方法时，检查`l.getArray()==expectedArray`：

```java
//数组引用本体
private final CopyOnWriteArrayList<E> l;
private final int offset;
private int size;
//拷贝时
private Object[] expectedArray;

COWSubList(CopyOnWriteArrayList<E> list,
           int fromIndex, int toIndex) {
    l = list;
    expectedArray = l.getArray();
    offset = fromIndex;
    size = toIndex - fromIndex;
}
private void checkForComodification() {
    if (l.getArray() != expectedArray)
        throw new ConcurrentModificationException();
}
```

### 场景

#### 优点

- 保留了读操作的高性能。
- 避免了并发修改抛出的ConcurrentModificationException异常。

#### 缺点

- 写操作太多时，将产生高内存消耗。因为需要拷贝出新数组。
- 读操作不能保证看到最新的数据，即使写操作已经开始执行了。因为直到写操作执行`setArray`之前，读操作都无法看到最新数据。

#### 适用场景

- 读操作多，写操作少的场景。
- 读操作允许看到非最新数据的场景。

### CopyOnWriteArraySet

CopyOnWriteArraySet内部实现是CopyOnWriteArrayList。

跟容器类Set不一样，不管是get、add、remove等等方法，性能都要低很多（毕竟底层是个List数组）。

Iterator的功能也跟CopyOnWriteArrayList相同，不提供修改的方法，并发修改时不会抛出异常。