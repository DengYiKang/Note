# ThreadLocal

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

### 总结

+ ThreadLocalMap是Thread类的成员变量，在ThreadLocal类中定义，由ThreadLocal类提供包装的set/get方法。
+ Thread通过ThreadLocal实例来访问对应的本地变量，本地变量由Thread和ThreadLocal来对应。
+ 第一次调用get时，如果map中没有对应的key，那么将会调用`initialValue`。理论上`initialValue`只会被调用一次，除非使用`remove`方法。
+ ThreadLocalMap是核心内部类，在它提供的set/get方法中，每次搜索到stale entry，都会对它为首的连续段进行清理。这种清理机制能保证待搜索的位置一定在以hash值下标为首的连续段中，提高了搜索效率。
+ 当所用量大于1/2时，将进行扩容。