# ThreadLocal

[TOC]

这个类提供线程本地变量。各个线程通过该类的get/set方法来访问线程隔离的变量。表面上看好像是ThreadLocal维护了一个变量表，存储着各个线程的同一个变量名的不同版本，事实上该变量是存储在各个线程内的map中的。一个线程得到一个ThreadLocal实例时，将ThreadLocal实例作为map中的key取访问本地的map，得到本地的ThreadLocal实例对应的变量。

## 例子

```java
//给各个线程标记id，不可改动（可以开放set方法来允许线程改动自身id） 
public class ThreadId {
     //提供初始化的数值
     private static final AtomicInteger nextId = new AtomicInteger(0);

     //ThreadLocal变量
     private static final ThreadLocal<Integer> threadId =
         new ThreadLocal<Integer>() {
             @Override protected Integer initialValue() {
                 return nextId.getAndIncrement();
         }
     };

     //get方法
     public static int get() {
         return threadId.get();
     }
 }
```

## 源码

### 成员变量

```java
//当前实例的hashCode，这个code是作为各个线程的threadLocals变量（ThreadLocalMap类型）的key
private final int threadLocalHashCode = nextHashCode();
//类变量，保证所有的ThreadLocal实例的hashCode不重复
private static AtomicInteger nextHashCode = new AtomicInteger();
//魔数，这个数能在2幂次容量的哈希表上能完美散列
private static final int HASH_INCREMENT = 0x61c88647;
private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
}
private void setThreshold(int len) {
	threshold = len * 2 / 3;
}
```

### initialValue

```java
//该方法默认返回null，需要实现
//当某个线程调用thread_local.get()时，如果key没有对应的值，那么将调用initialValue()方法
protected T initialValue() {
    return null;
}
```

### SuppliedThreadLocal

```java
static final class SuppliedThreadLocal<T> extends ThreadLocal<T> {

    private final Supplier<? extends T> supplier;

    SuppliedThreadLocal(Supplier<? extends T> supplier) {
        this.supplier = Objects.requireNonNull(supplier);
    }

    @Override
    protected T initialValue() {
        return supplier.get();
    }
}
```

对于实现初始值的提供方法，有两种方法：

1、用匿名内部类的方式

2、用SuppliedThreadLocal子类，向构造器提供`Supplier<? extends T> `接口

### get

```java
public T get() {
    //得到调用该方法的线程t
    Thread t = Thread.currentThread();
    //得到线程t的map
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        //key值是threadlocal实例
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    //如果不存在（即第一次调用），那么调用setInitialValue方法
    return setInitialValue();
}
//设置初始值（set(k, v)）
//如果线程的map为null，那么先创建一个map
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
//创建map
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

### set

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

### remove

```java
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);
}
```

### ThreadLocalMap

ThreadLocalMap是最核心的部分，在ThreadLocal类中定义，由ThreadLocal类来提供set/get方法，但作为Thread类的成员变量。

#### 成员变量

```java
//父类WeakReference的referent成员的类型为ThreadLocal<?>，作为key
static class Entry extends WeakReference<ThreadLocal<?>> {
    //ThreadLocal实例对应的value
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}

//初始容量，为2的幂
private static final int INITIAL_CAPACITY = 16;

//entry数组
private Entry[] table;

//entry的个数
private int size = 0;

//阈值，当size大于阈值时进行rehash
private int threshold; // Default to 0
```

#### 线性探测

```java
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}
private static int prevIndex(int i, int len) {
    return ((i - 1 >= 0) ? i - 1 : len - 1);
}
```

探测函数使得table为一个环形数组。

#### 构造器

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    //当len为2的幂次方时，x&(len-1)=x%len
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}

/**
 * 由paranetMap生成本地map
 * 将parentMap中非null且key也非null的entry进行存储
 * 注意，parentMap与本地map的变量下标不一定一一对应
 * 本地map的结构要优于parentMap
 * （本地map相同的key存放的下标可能比parentMap的小，因为parentMap中可能存在key为null的entry）
 */
private ThreadLocalMap(ThreadLocalMap parentMap) {
    Entry[] parentTable = parentMap.table;
    int len = parentTable.length;
    setThreshold(len);
    table = new Entry[len];

    for (int j = 0; j < len; j++) {
        Entry e = parentTable[j];
        if (e != null) {
            @SuppressWarnings("unchecked")
            ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
            if (key != null) {
                //childValue为一个变换函数，用户可自定义
                Object value = key.childValue(e.value);
                Entry c = new Entry(key, value);
                int h = key.threadLocalHashCode & (len - 1);
                while (table[h] != null)
                    h = nextIndex(h, len);
                table[h] = c;
                size++;
            }
        }
    }
}
//该方法需要实现，做为一个变换函数，注意这个方法不是ThreadLocalMap类的方法
T childValue(T parentValue) {
	throw new UnsupportedOperationException();
}
```

#### get

```java
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    //第一次探测到就直接返回
    if (e != null && e.get() == key)
        return e;
    else
        //线性探测，同时整理entry数组
        return getEntryAfterMiss(key, i, e);
}
//在第一次miss后，线性探测，同时整理table
private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            //如果当前slot为stale的（非null，但key为null），那么整理以下标i为首的非null连续段
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```

```java
//staleSlot对应的entry不为null，但是key为null
//整理对象是从staleSlot开始的非null连续段
//将stale entry（非null、key为null）置null
//如果entry的下标与计算的hash值冲突，那么清空它，为它重新寻找新的位置（新位置一定≤旧位置）
//(因为前面的被清理的staleSlot可能恰好作为新的位置)
//返回staleSlot往后的第一个null entry的下标
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;

    // 将staleSlot置null
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;

    Entry e;
    int i;
    for (i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        if (k == null) {
            //将stale entry（非null、key为null）置null
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            //如果entry的下标与计算的hash值冲突，那么清空它，为它重新寻找新的位置
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
```

#### set

```java
private void set(ThreadLocal<?> key, Object value) {


    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        //找到了，直接替换新的值
        if (k == key) {
            e.value = value;
            return;
        }

        //如果发现了一个staleSlot，那么进行replaceStaleEntry
        //replaceStaleEntry在进行set的同时对
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    tab[i] = new Entry(key, value);
    int sz = ++size;
    //如果没有清理掉一个stale entry，且entry使用的数量超出阈值了，那么需要rehash（扩容）
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```

```java
private void replaceStaleEntry(ThreadLocal<?> key, Object value,
                               int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;
    Entry e;

    //slotToExpunge标记包含staleSlot在内的非null连续段中的第一个stale slot（不包括staleSlot本身）
    //搜索完成后，如果slotToExpunge==staleSlot，那么这个连续段只有staleSlot一个stale entry，就不需要清理了
    //（最终（k，v）肯定在staleSlot对应的位置上的）
    int slotToExpunge = staleSlot;
    //从staleSlot向前搜索
    for (int i = prevIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = prevIndex(i, len))
        if (e.get() == null)
            slotToExpunge = i;

    //从staleSlot向后搜索
    //完成以下任务：
    //1、维护slotToExpunge变量
    //2、如果在staleSlot往后的非null连续段找到key的位置i，那么将这个key对应的entry跟staleSlot交换，之后清理
    //下标i往后的非null连续段(因为被交换过了，所以这时下标i的位置是stale entry)
    //3、如果没有找到，那么将（k，v）放在staleSlot的位置上，然后判断是否需要清理(slotToExpunge==staleSlot)
    for (int i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
		//找到了，那么就将当前位置的entry与staleSlot交换，并对value进行更新
        if (k == key) {
            e.value = value;

            tab[i] = tab[staleSlot];
            tab[staleSlot] = e;

            //如果当前还没有搜索到除了staleSlot之外的stale entry，那么将slotToExpunge设置为i（交换后i位置是stale的）
            //如果slotToExpunge != staleSlot，之前肯定搜索到了stale entry，不用更新slotToExpunge
            if (slotToExpunge == staleSlot)
                slotToExpunge = i;
            //清理
            cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
            return;
        }

		//找到了除staleSlot的第一个stale entry
        if (k == null && slotToExpunge == staleSlot)
            slotToExpunge = i;
    }

    //如果没有找到key，那么直接把（k，v）放在staleSlot对应的位置
    tab[staleSlot].value = null;
    tab[staleSlot] = new Entry(key, value);

    //如果有其他的stale entry在该连续段里，那么进行清理
    if (slotToExpunge != staleSlot)
        cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
}
```

```java
//从i开始清理
//搜索长度为log2(n)
//每次碰到一个stale entry，那么调用expungeStaleEntry，将stale entry所在的连续段进行清理，然后重置搜索长度
//返回是否执行了清理
private boolean cleanSomeSlots(int i, int n) {
    boolean removed = false;
    Entry[] tab = table;
    int len = tab.length;
    do {
        i = nextIndex(i, len);
        Entry e = tab[i];
        if (e != null && e.get() == null) {
            n = len;
            removed = true;
            i = expungeStaleEntry(i);
        }
    } while ( (n >>>= 1) != 0);
    return removed;
}
```

```java
private void rehash() {
    //全面清理
    expungeStaleEntries();

    //不等式右侧=len/2
    //使用的量大于1/2时进行resize
    if (size >= threshold - threshold / 4)
        resize();
}
//全面清理，从头到位expungeStaleEntry
private void expungeStaleEntries() {
    Entry[] tab = table;
    int len = tab.length;
    for (int j = 0; j < len; j++) {
        Entry e = tab[j];
        if (e != null && e.get() == null)
            expungeStaleEntry(j);
    }
}
//两倍扩容，重新hash（因为与运算的对象是len的表达式，会变动）
private void resize() {
    Entry[] oldTab = table;
    int oldLen = oldTab.length;
    int newLen = oldLen * 2;
    Entry[] newTab = new Entry[newLen];
    int count = 0;

    for (int j = 0; j < oldLen; ++j) {
        Entry e = oldTab[j];
        if (e != null) {
            ThreadLocal<?> k = e.get();
            if (k == null) {
                e.value = null; // Help the GC
            } else {
                int h = k.threadLocalHashCode & (newLen - 1);
                while (newTab[h] != null)
                    h = nextIndex(h, newLen);
                newTab[h] = e;
                count++;
            }
        }
    }

    setThreshold(newLen);
    size = count;
    table = newTab;
}
```

## 总结

+ ThreadLocalMap是Thread类的成员变量，在ThreadLocal类中定义，由ThreadLocal类提供包装的set/get方法。
+ Thread通过ThreadLocal实例来访问对应的本地变量，本地变量由Thread和ThreadLocal来对应。
+ 第一次调用get时，如果map中没有对应的key，那么将会调用`initialValue`。理论上`initialValue`只会被调用一次，除非使用`remove`方法。
+ ThreadLocalMap是核心内部类，在它提供的set/get方法中，每次搜索到stale entry，都会对它为首的连续段进行清理。这种清理机制能保证待搜索的位置一定在以hash值下标为首的连续段中，提高了搜索效率。
+ 当所用量大于1/2时，将进行扩容。

## 面试问题

### ThreadLocal的用途

- ThreadLocal用来给各个线程提供线程隔离的局部变量。使用很简单，通过调用同一个ThreadLocal对象的get/set方法来读写这个ThreadLocal对象对应的value，但是线程A set值后，不会影响到线程B之后get到的值。
- ThreadLocal对象通常是`static`的，因为它在map里作为key使用，所以在各个线程中需要复用。

### 简单说下ThreadLocal的实现原理

- 每个线程在运行过程中都可以通过`Thread.currentThread()`获得与之对应的Thread对象，而每个Thread对象都有一个`ThreadLocalMap`类型的成员，`ThreadLocalMap`是一种hashmap，它以ThreadLocal作为key。
- 所以，只有通过Thread对象和ThreadLocal对象二者，才可以唯一确定到一个value上去。线程隔离的关键，正是因为这种对应关系用到了Thread对象。
- 线程可以根据自己的需要往`ThreadLocalMap`里增加键值对，当线程从来没有使用到ThreadLocal时（指调用get/set方法），Thread对象不会初始化这个`ThreadLocalMap`类型的成员。

### 讲讲ThreadLocalMap这个Map

- 它是一种特殊实现的HashMap实现，它必须以ThreadLocal类型作为key。
- 容量必为2的幂，使得它，可通过位与操作得到数组下标。
- 在解决哈希冲突时，使用开放寻址法（索引往后移动一位）和环形数组（索引移动到length时，跳转到0）。这样，只有size到达threshold时，才会resize操作。

### ThreadLocal作为key，它的哈希值怎么计算的？

- 利用魔数`0x61c88647`从0递增，得到每个ThreadLocal对象的哈希值。
- 两个线程同时构造ThreadLocal对象，也能保证它俩的哈希值不同，因为利用了AtomicInteger。
- 利用魔数`0x61c88647`的好处在于，这样得到的哈希值再取模得到下标，下标是均匀分布的。而这又可能带了另一个好处：当哈希冲突时，大概率能更快找到可以放置的位置。
- 不要被魔数`0x61c88647`的网上示例迷惑，示例通常将魔数递增16次，再将每次递增的结果取模16，发现16次取模的结果（0-15）都不一样。这一点确实有点神奇，它利用了斐波那契数列（和黄金分割数），但是你把魔数定为`0x01`，一样能实现16次取模的结果（0-15）都不一样。重点在于，它能均匀分布。

### 为什么Entry继承了WeakReference？

- 首先，WeakReference的父类成员referent，如果referent指向的对象没有强引用指着它，那么referent指向的对象就可能被回收，从而使得referent引用为null。
- 而referent成员是作为key来使用的，这样key为null的entry（称为stale entry）在get/set操作中**可能**会被间接清理掉。
- 所以，继承WeakReference的原因是为了能更快回收资源，但前提是：
    - 没有强引用指向ThreadLocal对象。
    - 且jvm执行了gc，回收了ThreadLocal对象，出现了stale entry。
    - 且之后get/set操作的间接调用刚好清理掉了这个stale entry。
- 综上得知，要想通过WeakReference来获得更快回收资源的好处，其实比较难。所以，当你知道当前线程已经不会使用这个ThreadLocal对应的值时，显式调用`remove`将是最佳选择。

### Entry继承了WeakReference，可能造成内存泄漏？

- 首先要知道，这一点并不是WeakReference的锅。
- 一般情况下，ThreadLocal对象都会设置成`static`域，它的生命周期通常和一个类一样长。
- 当一个线程不再使用ThreadLocal读写值后，如果不调动`remove`，这个线程针对该ThreadLocal设置的value对象就已经内存泄漏了。且由于ThreadLocal对象生命周期一般很长，现在Entry对象、它的referent成员、它的value成员三者都内存泄漏。
- 而Entry继承了WeakReference，反而降低了内存泄漏的可能性（见上一问题）。
- 综上得知，内存泄漏不是因为继承了WeakReference，而且因为ThreadLocal对象生命周期一般很长，且使用完毕ThreadLocal后，线程没有主动调用`remove`。

### 线程池中的线程使用ThreadLocal需要注意什么？

- 由于ThreadLocalMap是Thread对象的成员，当对应线程运行结束销毁时，自然这个ThreadLocalMap类型的成员也会被回收。
- 但如果想依赖上面这点来避免内存泄漏，就大错特错了。因为线程池里的线程为了复用线程，一般不会直接销毁掉完成了任务的线程，以下一次复用。
- 所以，线程使用完毕ThreadLocal后，主动调用`remove`来避免内存泄漏，才是万全之策。
- 另外，线程池中的线程使用完毕ThreadLocal后，不主动调用`remove`，还可能造成：get值时，get到上一个任务set的值，直接造成程序错误。

### 使用ThreadLocal有什么好处？（可以解决什么问题？）

- 相比`synchronized`使用锁从而使得多个线程可以安全的访问同一个共享变量，现在可以直接转换思路，让线程使用自己的私有变量，直接避免了并发访问的问题。
- 当一个数据需要传递给某个方法，而这个方法处于方法调用链的中间部分，那么如果采用加参数传递的方式，势必为影响到耦合性。而如果使用ThreadLocal来为线程保存这个数据，就避免了这个问题，而且对于这个线程，它在任何时候都可以取到这个值。

