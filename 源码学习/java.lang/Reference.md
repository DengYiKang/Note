# Reference

[TOC]

## 简介

Java中有四种引用，这四种引用从高到低分别为：

+ StrongReference

	这个引用没有相应的类与之对应，但是强引用比较普遍如：`Object obj=new Object();`，这里的obj就是强引用。如果一个对象持有强引用，那么垃圾回收器不会回收该对象，当内存不足时情愿抛出OOM异常也不会回收。

+ SoftReference

	如果一个对象只有软引用，那么在内存充足的情况下是不会回收此对象的，但是，在内存不足的情况下就会回收此对象。

	```java
	public class TestReference3 {
	        private static ReferenceQueue<Object> rq = new ReferenceQueue<Object>();
	        public static void main(String[] args){
	            Object obj = new Object();
	            SoftReference<Object> sf = new SoftReference(obj,rq);
	            //输出true
	            System.out.println(sf.get()!=null);
	            System.gc();
	            obj = null;
	            //输出true
	            //内存充足的情况下，gc后仍然没有被回收
	            System.out.println(sf.get()!=null);
	        }
	    }
	```

+ WeakReference

	当gc发现一个对象只有弱引用时，就会回收此对象。但要注意gc所在的线程优先级比较低，不会立即发现对象并回收。

	```java
	    public class TestWeakReference {
	        private static ReferenceQueue<Object> rq = new ReferenceQueue<Object>();
	        public static void main(String[] args) {
	            Object obj = new Object();
	            WeakReference<Object> wr = new WeakReference(obj,rq);
	            System.out.println(wr.get()!=null);
	            //obj=null后，强引用没了，只剩下弱引用
	            obj = null;
	            System.gc();
	            System.out.println(wr.get()!=null);//false，这是因为WeakReference被回收
	        }
	    }
	```

+ PhantomReference

	虚引用不会影响对象的生命周期。虚引用的作用为：跟踪垃圾回收过程，在对象被收集器回收时收到一个系统通知。

	当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在垃圾回收后，将这个虚引用加入引用队列，在其关联的虚引用出队前，不会彻底销毁该对象。 所以可以通过检查引用队列中是否有相应的虚引用来判断对象是否已经被回收了。

	与软引用和弱引用不同，显式使用虚引用可以阻止对象被清除，只有在程序中显式或者隐式移除这个虚引用时，这个已经执行过finalize方法的对象才会被清除。想要显式的移除虚引用的话，只需要将其从引用队列中取出然后扔掉（置为null）即可。

	```java
	public class PhantomReferenceTest {
	    private static final List<Object> TEST_DATA = new LinkedList<>();
	    private static final ReferenceQueue<TestClass> QUEUE = new ReferenceQueue<>();
	
	    public static void main(String[] args) {
	        TestClass obj = new TestClass("Test");
	        PhantomReference<TestClass> phantomReference = new PhantomReference<>(obj, QUEUE);
	
	        // 该线程不断读取这个虚引用，并不断往列表里插入数据，以促使系统早点进行GC
	        new Thread(() -> {
	            while (true) {
	                TEST_DATA.add(new byte[1024 * 100]);
	                try {
	                    Thread.sleep(1000);
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                    Thread.currentThread().interrupt();
	                }
	                System.out.println(phantomReference.get());
	            }
	        }).start();
	
	        // 这个线程不断读取引用队列，当弱引用指向的对象被回收时，该引用就会被加入到引用队列中
	        new Thread(() -> {
	            while (true) {
	                Reference<? extends TestClass> poll = QUEUE.poll();
	                if (poll != null) {
	                    System.out.println("--- 虚引用对象被jvm回收了 ---- " + poll);
	                    System.out.println("--- 回收对象 ---- " + poll.get());
	                }
	            }
	        }).start();
	
	        obj = null;
	
	        try {
	            Thread.currentThread().join();
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	            System.exit(1);
	        }
	    }
	
	    static class TestClass {
	        private String name;
	
	        public TestClass(String name) {
	            this.name = name;
	        }
	
	        @Override
	        public String toString() {
	            return "TestClass - " + name;
	        }
	    }
	}
	```

	```shell
[GC (Allocation Failure)  1024K->432K(3584K), 0.0113386 secs]
	[GC (Allocation Failure)  1455K->520K(3584K), 0.0133610 secs]
	[GC (Allocation Failure)  1544K->648K(3584K), 0.0008654 secs]
	null
	null
	null
	[GC (Allocation Failure)  1655K->973K(3584K), 0.0008111 secs]
	null
	...省略几个null的输出
	[GC (Allocation Failure)  1980K->1997K(3584K), 0.0009289 secs]
	[Full GC (Ergonomics)  1997K->1870K(3584K), 0.0048483 secs]
	--- 弱引用对象被jvm回收了 ---- java.lang.ref.PhantomReference@74cbe23d
	--- 回收对象 ---- null
	null
	...省略几个null和几次Full GC的输出
	[Full GC (Ergonomics)  2971K->2971K(3584K), 0.0024850 secs]
	[Full GC (Allocation Failure)  2971K->2971K(3584K), 0.0022460 secs]
	Exception in thread "Thread-0" java.lang.OutOfMemoryError: Java heap space
		at weakhashmap.PhantomReferenceTest.lambda$main$0(PhantomReferenceTest.java:20)
		at weakhashmap.PhantomReferenceTest$$Lambda$1/2065951873.run(Unknown Source)
		at java.lang.Thread.run(Thread.java:748)
	```
	
	可以发现，虚引用对象被gc后并没有完全销毁，而是进入到引用队列。引用队列将其出队才算真正的销毁。
	
	使用场景：
	
	使用虚引用的目的就是为了得知对象被gc的时机，可以利用虚引用来进行销毁前的一些操作，比如资源释放等。
	
	虚引用有一个很重要的用途就是用来做堆外内存的释放，DirectByteBuffer就是通过虚引用来实现堆外内存的释放的。

## 源码

### Reference

#### 分析

Reference是SoftReference、WeakReference的基类。

一个Reference实例有四种状态：

+ Active：

	新创建的实例为Active的。可以转换成Pending状态或者Inactive状态。如果注册了队列，那么将该实例加入pending队列，并转换成Pending状态，否则直接转换成Inactive状态。

+ Pending:

	在pending队列中的实例的状态。当前实例等待入队（注意这个入队的队列不是pending队列，是指注册队列）。

+ Enqueued:

	在注册队列中。当被从注册队列移除时，将转成Inactive状态。

+ Inactive:

	当一个实例成Inactive状态后，状态将永不可变。

```java
private T referent;         /* Treated specially by GC */

//注册队列
volatile ReferenceQueue<? super T> queue;

/* When active:   NULL
 *     pending:   this
 *    Enqueued:   next reference in queue (or this if last)
 *    Inactive:   this
 *  因此可以通过next域来判断当前实例是否为active或者Enqueued
 */
@SuppressWarnings("rawtypes")
volatile Reference next;

/* When active:   next element in a discovered reference list maintained by GC (or this if last)
 *     pending:   next element in the pending list (or null if last)
 *   otherwise:   NULL
 */
transient private Reference<T> discovered;  /* used by VM */
```

可以注意到，遍历active与pending的实例是有vm通过discovered域来搜索的；遍历注册队列的实例是根据next域来搜索的。

```java
/*
* 每次回收都需要获取锁
*/
static private class Lock { }
private static Lock lock = new Lock();
```

```java
//注意这是类变量，所有在pending状态的实例都可以由该变量来搜索到
//该变量类似head结点，而next就是discovered域
private static Reference<Object> pending = null;
```

```java
Reference(T referent) {
    this(referent, null);
}

Reference(T referent, ReferenceQueue<? super T> queue) {
    this.referent = referent;
    this.queue = (queue == null) ? ReferenceQueue.NULL : queue;
}
```

ReferenceQueue类有两个类变量：

```java
static ReferenceQueue<Object> NULL = new Null<>();
static ReferenceQueue<Object> ENQUEUED = new Null<>();
```

NULL表示没有注册队列，ENQUEUED表示已经入队。这两个变量作为Reference类的成员变量queue的可选值。当Reference类实例入队注册队列时，queue将会赋值为ENQUEUED。

```java
//高优先级线程，处理pending状态的实例，将它们由cleaner处理或者入队注册队列
private static class ReferenceHandler extends Thread {

    private static void ensureClassInitialized(Class<?> clazz) {
        try {
            Class.forName(clazz.getName(), true, clazz.getClassLoader());
        } catch (ClassNotFoundException e) {
            throw (Error) new NoClassDefFoundError(e.getMessage()).initCause(e);
        }
    }

    static {
        // pre-load and initialize InterruptedException and Cleaner classes
        // so that we don't get into trouble later in the run loop if there's
        // memory shortage while loading/initializing them lazily.
        ensureClassInitialized(InterruptedException.class);
        ensureClassInitialized(Cleaner.class);
    }

    ReferenceHandler(ThreadGroup g, String name) {
        super(g, name);
    }

    public void run() {
        while (true) {
            tryHandlePending(true);
        }
    }
}

//处理pending队列
//一次处理一个
static boolean tryHandlePending(boolean waitForNotify) {
    Reference<Object> r;
    Cleaner c;
    try {
        synchronized (lock) {
            if (pending != null) {
                r = pending;
                //instanceof可能会导致oom，因此要在unlink之前执行
                //否则unlink后如果出现了oom，则无法回复
                c = r instanceof Cleaner ? (Cleaner) r : null;
                //unlink
                pending = r.discovered;
                r.discovered = null;
            } else {
                //wait可能导致oom
                //因为它可能会尝试产生异常对象
                if (waitForNotify) {
                    lock.wait();
                }
                // retry if waited
                return waitForNotify;
            }
        }
    } catch (OutOfMemoryError x) {
        // Give other threads CPU time so they hopefully drop some live references
        // and GC reclaims some space.
        // Also prevent CPU intensive spinning in case 'r instanceof Cleaner' above
        // persistently throws OOME for some time...
        Thread.yield();
        // retry
        return true;
    } catch (InterruptedException x) {
        // retry
        return true;
    }

    // Fast path for cleaners
    if (c != null) {
        c.clean();
        return true;
    }

    ReferenceQueue<? super Object> q = r.queue;
    //如果注册了队列，那么入队
    if (q != ReferenceQueue.NULL) q.enqueue(r);
    return true;
}
```

#### 总结

+ 一个Reference实例有四种状态：Active、Pending、Enqueued、Inactived
+ Reference的成员变量queue保存注册队列的引用。如果无，则为ReferenceQueue.NULL，如果已经入队，则为ReferenceQueue.ENQUEUED。
+ VM根据discovered域来搜索active实例
+ ReferenceHandler根据discovered域来搜索pending实例
+ ReferenceQueue根据next域来搜索enqueued实例

