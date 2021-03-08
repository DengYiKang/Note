# CompletableFuture

### 前言

我们知道`FutureTask`实现了task异步执行，但对于执行结果的获取，如果异步执行还在进行中，那么线程只能get阻塞等待，或者轮询`sDone`，这两种方式都和我们开始实现异步的初衷相违背。所以就诞生了这个`CompletableFuture`，它的最大不同之处在于，通过提供回调函数的概念，把处理执行结果的过程也放到异步线程里去做。

### 基础设施

`CompletableFutur`实现了`Future`接口和`CompletionStage`接口，`CompletionStage`接口提供了很多异步回调的函数。

#### 创建CompletableFuture

有两种方法可以创建`CompletableFuture`：

+ 静态方法，比如`supplyAsync`。属于零输入，执行时机是马上执行。
+ 成员方法，比如`CompletableFuture对象.thenApply`。属于有输入，执行时机是调用对象的完成时机。

#### CompletableFuture成员

```java
volatile Object result;       // Either the result or boxed AltResult
volatile Completion stack;    // Top of Treiber stack of dependent actions
```

`CompletableFuture`是在用户使用过程中唯一能直接接触到的对象。

- `result`存放执行结果，正常结果或者抛出的异常都要存放，所以是Object。任务执行完毕后，`result`会变成非null。
- `stack`是一个链栈，存放与`this`对象直接关联的`Completion`对象。`Completion`对象是用来驱动某一个`CompletableFuture`对象，所谓的驱动，就是使得这个`CompletableFuture`对象的`result`成员变为非null。

#### Completion内部类

`Completion`对象是用户接触不到的，它用来驱动`CompletableFuture`对象。

```java
abstract static class Completion extends ForkJoinTask<Void>
    implements Runnable, AsynchronousCompletionTask {
    ......
}
```

- 它继承了`ForkJoinTask`，但也仅仅是为了套上`ForkJoinTask`的壳子，因为`CompletableFuture`默认的线程池是`ForkJoinPool.commonPool()`。
- 它也实现了`Runnable`，这使得它也能被一个普通线程正常执行。
- Completion有很多继承的子类，它们分别实现了`tryFire`方法。

#### AltResult内部类

```java
static final class AltResult { 
    final Throwable ex;        
    AltResult(Throwable x) { this.ex = x; }
}
static final AltResult NIL = new AltResult(null);
```

前面提到，任务执行完毕后，`result`会变成非null。但如果执行结果就是null该怎么办。所以用这个对象来包装一下null。

#### Signaller内部类

```java
static final class Signaller extends Completion
    implements ForkJoinPool.ManagedBlocker {
    long nanos;                    // wait time if timed
    final long deadline;           // non-zero if timed
    volatile int interruptControl; // > 0: interruptible, < 0: interrupted
    volatile Thread thread;
    ......
}
```

配合get或者join使用的，实现对 想获取执行结果的线程 的阻塞和唤醒的功能。

### 从supplyAsync + thenApply(thenApplyAsync)理解

`CompletableFuture`实现了`CompletionStage`，代表一个执行阶段，我们可以在执行阶段之后添加后续任务，当前一个执行阶段完毕时，马上触发后续任务。

```java
public static void test() {
        CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
            String supplyAsyncResult = " "+Thread.currentThread().getName()+" Hello world! ";
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(supplyAsyncResult);
            return supplyAsyncResult;
        }).thenApply(r -> {  //添加后续任务
            String thenApplyResult = Thread.currentThread().getName()+r + " thenApply! ";
            System.out.println(thenApplyResult);
            return thenApplyResult;
        });

        try {
            System.out.println(completableFuture.get() + " finish!");
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
/*output:
 ForkJoinPool.commonPool-worker-9 Hello world! 
ForkJoinPool.commonPool-worker-9 ForkJoinPool.commonPool-worker-9 Hello world!  thenApply! 
ForkJoinPool.commonPool-worker-9 ForkJoinPool.commonPool-worker-9 Hello world!  thenApply!  finish!
*/
```

首先注意到这是一种链式编程，`supplyAsync`返回的是一个`CompletableFuture`对象（代表一个执行阶段），然后在这个`CompletableFuture`对象上再执行`thenApply`，又返回了一个新的`CompletableFuture`对象（代表下一个执行阶段）。而且发现，两个task都是在另外的线程里执行的，这完全实现了异步处理的效果。

为了方便称呼，我们叫第一个task为前一个stage，第二个task为当前stage。

本文也会把`CompletableFuture`对象称为一个stage。

#### supplyAsync

```java
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
    return asyncSupplyStage(asyncPool, supplier);
}

static <U> CompletableFuture<U> asyncSupplyStage(Executor e,
                                                 Supplier<U> f) {
    if (f == null) throw new NullPointerException();
    CompletableFuture<U> d = new CompletableFuture<U>();
    e.execute(new AsyncSupply<U>(d, f));
    return d;
}
```

可见这个`CompletableFuture`对象是new出来以后就直接返回的，但是刚new的`CompletableFuture`对象的`result`成员是为null，因为task还没有执行完。而task的执行交给了`e.execute(new AsyncSupply(d, f))`。

```java
static final class AsyncSupply<T> extends ForkJoinTask<Void>
        implements Runnable, AsynchronousCompletionTask {
    CompletableFuture<T> dep; Supplier<T> fn;
    AsyncSupply(CompletableFuture<T> dep, Supplier<T> fn) {
        this.dep = dep; this.fn = fn;
    }

    public final Void getRawResult() { return null; }
    public final void setRawResult(Void v) {}
    public final boolean exec() { run(); return true; }

    public void run() {
        CompletableFuture<T> d; Supplier<T> f;
        if ((d = dep) != null && (f = fn) != null) {
            dep = null; fn = null; //为了防止内存泄漏，方便GC.同时dep为null也是一种代表当前Completion对象的关联stage已完成的标志
                if (d.result == null) {
            if (d.result == null) {
                try {
                    d.completeValue(f.get());//执行task
                } catch (Throwable ex) {//执行task期间抛出了异常
                    d.completeThrowable(ex);
                }
            }
            d.postComplete();
        }
    }
}
```

很显然，为了能够`e.execute`，`AsyncSupply`也必须是一个`Runnable`对象。执行`e.execute(new AsyncSupply<U>(d, f))`，run函数就会被另一个线程执行。当task被异步执行完毕后，会调用`completeValue`或`completeThrowable`来为result成员赋值。

![在这里插入图片描述](../../../pic/40)

上图体现了`supplyAsync()`的过程，对于调用者来说，只能接触到stage对象，并且调用者根本不知道stage对象何时能产生运行结果。对于实现来说，把task包装成一个`AsyncSupply`对象，另起线程执行task，执行完毕后为stage对象赋值运行结果。

注意，stage完成的标志，就是它的`result`成员非null。

#### thenApply(thenApplyAsync)

在`supplyAsync`直接返回了个`CompletableFuture`对象后，主线程在这个对象上调用`thenApply`或`thenApplyAsync`将后续stage接续到前一个stage的后面。

```java
public <U> CompletableFuture<U> thenApply(
    Function<? super T,? extends U> fn) {
    return uniApplyStage(null, fn);
}

public <U> CompletableFuture<U> thenApplyAsync(
    Function<? super T,? extends U> fn) {
    return uniApplyStage(asyncPool, fn);
}
```

`thenApply`不会传入`Executor`，因为它优先让当前线程（例子中是main线程）来执行后续stage的task。具体的说：

- 当发现前一个stage已经执行完毕时，直接让当前线程来执行后续stage的task。
- 当发现前一个stage还没执行完毕时，则把当前stage包装成一个`UniApply`对象，放到前一个stage的栈中。执行前一个stage的线程，执行完毕后，接着执行后续stage的task。
- 总之，要么是一个异步线程走到底，要么让当前线程来执行后续stage（因为异步线程已经结束，而且你又没有给`Executor`，那只好让当前线程来执行咯）。

`thenApplyAsync`会传入一个`Executor`，因为它总是让Executor线程池里面的线程 来执行后续stage的task。具体的说：

+ 当发现前一个stage已经执行完毕时，直接让`Executor`来执行。
+ 当发现前一个stage还没执行完毕时，则等到执行前一个stage的线程执行完毕后，再让`Executor`来执行。
+ 总之，无论哪种情况，执行后一个stage的线程肯定不是当前添加后续stage的线程（例子中是main线程）了。

```java
private <V> CompletableFuture<V> uniApplyStage(
    Executor e, Function<? super T,? extends V> f) {
    if (f == null) throw new NullPointerException();
    CompletableFuture<V> d =  new CompletableFuture<V>();
    //如果e不为null，说明当前stage是无论如何都需要被异步执行的。所以短路后面的d.uniApply。
    //如果e为null，说明当前stage是可以允许被同步执行的。所以需要尝试一下d.uniApply。
    if (e != null || !d.uniApply(this, f, null)) {
        //进入此分支有两种情况：
    	//1. 要么e不为null，前一个stage不一定执行完毕。就算前一个stage已经执行完毕，还可以用e来执行当前stage
    	//2. 要么e为null，但前一个stage还没执行完毕。所以只能入栈等待
        UniApply<T,V> c = new UniApply<T,V>(e, d, this, f);
        push(c);
        //（考虑e为null）入栈后需要避免，入栈后刚好前一个stage已经执行完毕的情况。这种特殊情况，如果不执行c.tryFire(SYNC)，当前stage永远不会完成。
        //（考虑e不为null）入栈后需要避免，入栈前 前一个stage已经执行完毕的情况。
        //下面这句，有可能发现前一个stage已经执行完毕，然后马上执行当前stage
        c.tryFire(SYNC);
    }
    return d;
}
```

+ 从`CompletableFuture<V> d = new CompletableFuture<V>()`和`return d`来看，还是和之前一样，new出来一个`CompletableFuture`对象后就尽快返回。
+ 如果`Executor e`为null（当前stage是可以允许被同步执行的），并且此时前一个stage已经结束了，这种情况应该让当前线程来同步执行当前stage。但我们其实不知道前一个stage是否结束，所以通过`d.uniApply(this, f, null)`检测前一个stage是否已经结束。如果`d.uniApply(this, f, null)`返回true，说明发现了前一个stage已经结束，并且当前线程执行完毕当前stage，所以这种情况就会直接`return d`。
	+ `d.uniApply(this, f, null)`的第三个实参为null，这代表与当前stage相关联的`Completion`对象还没有入栈（还没`push(c)`），即不可能有别的线程与当前线程来竞争执行当前stage。这样`d.uniApply(this, f, null)`里面的逻辑就变简单了，要么发现前一个stage还没执行完，直接返回false；要么发现前一个stage执行完毕，那么执行当前stage后，返回true。

进入分支有两种情况：

+ 如果`e`不为null：
	+ 如果前一个stage已经执行完毕：当前线程在`c.tryFire(SYNC)`中把接管的当前stage转交给`e`执行。
	+ 如果前一个stage还没执行完毕：当前线程会直接返回，等到执行前一个stage的线程来把当前stage转交给`e`执行。
+ 如果`e`为null：
	- 并且前一个stage还没执行完毕。
+ 上面几种情况，最终都会入栈，不管`e`是否为null，都有必要再尝试一下`c.tryFire(SYNC)`，避免此时前一个stage已经完成的情况。
+ `c.tryFire(SYNC)`中也会执行类似`d.uniApply(this, f, null)`，而且你会发现两种调用环境，`uniApply`成员函数的`this`对象是一样的（当前stage），第一个实参是一样的（前一个stage），第二个实参也是同一个函数式接口对象，只有第三个实参不一样。

#### UniApply内部类#tryFire

在讲`tryFire`之前，我们先看看`tryFire`有几处调用：

+ `uniApplyStage`中的同步调用，`c.tryFire(SYNC)`。

+ 执行前一个stage的线程，在`run`的`d.postComplete()`中，会调用`tryFire(NESTED)`。

+ 上面两处，`tryFire`的this对象都是我们分析过程提到的当前stage。并且，这说明`tryFire`可能会有多线程的竞争问题，来看看`tryFire`是怎么解决的。

	+ 多线程竞争，比如当前线程入栈后，执行前一个stage的线程刚完事，正要触发后续stage（`run`的`d.postComplete()`中）。

	```java
	//src代表前一个stage, dep代表当前stage。  UniApply对象将两个stage组合在一起了。
	static final class UniApply<T,V> extends UniCompletion<T,V> {
	    Function<? super T,? extends V> fn;
	    UniApply(Executor executor, CompletableFuture<V> dep,
	             CompletableFuture<T> src,
	             Function<? super T,? extends V> fn) {
	        super(executor, dep, src); this.fn = fn;
	    }
	    final CompletableFuture<V> tryFire(int mode) {
	        CompletableFuture<V> d; CompletableFuture<T> a;
	        //1. 如果dep为null，说明当前stage已经被执行过了
			//2. 如果uniApply返回false，说明当前线程无法执行当前stage。返回false有可能是因为
			//     1. 前一个stage没执行完呢
			//     2. 前一个stage执行完了，但当前stage已经被别的线程执行了。如果提供了线程池，那么肯定属于被别的线程执行了。   
	        if ((d = dep) == null ||
	            !d.uniApply(a = src, fn, mode > 0 ? null : this))
	            return null;
	        //执行到这里，说明dep不为null，而且uniApply返回true，说明当前线程执行了当前stage
	        dep = null; src = null; fn = null;
	        return d.postFire(a, mode);
	    }
	}
	```

太难了，看不懂了，感觉套来套去，很乱orz。。。以后再跟着学吧 [博客](https://blog.csdn.net/anlian523/article/details/108023786)。