# Runnable、Callabe、FutureTask使用浅析

### Runnable

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

创建线程有两种方式：

+ 重写`Thread`的`run`方法：

	```java
	new Thread() {
	            @Override
	            public void run() {
	                int count = 0;
	                for(int i = 1;i <= 100;i++) 
	                    count += i;
	            }
	        }.start();
	```

	调用`Thread#start()`后，会创建一个新线程来执行这个`Thread`的run方法。但上面这种执行者和执行task绑定在一起了，不灵活。

+ 传递一个`Runnable`对象给`Thread`：

	```java
	new Thread(new Runnable(){
	            @Override
	            public void run() {
	                int count = 0;
	                for(int i = 1;i <= 100;i++)
	                    count += i;
	            }
	        }).start();
	```

	这样，通过创建一个`Runnable`匿名内部类对象，可以达到同样的效果，但是却把执行者和执行task分开了。

### Callable

与Runnable相比，Callable接口有些不同之处：

+ Runnable接口没有返回值，Callable接口有返回值。又因为是返回值是泛型，所以任何类型的返回值都支持。
+ Runnable接口的run方法没有throws Exception。这意味着，Runnable不能抛出异常（子类不能抛出比父类更多的异常，但现在Runnable的run方法本身没有抛出任何异常）；Callable接口可以抛出异常。

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

但注意，`Thread`并没有一个构造器可以接受`Callable`参数的，而且`Thread`也没有一个`Callable`类型成员的。

`Thread`只有接受`Runnable`参数的构造器。

```java
   public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }
```

因此想要使用`Callable`还需要依靠其他东西。

### FutureTask

要想使用`Callable`还得需要依靠`FutureTask`。

![](../pic/38)

```java
 public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }
```

`FutureTask`的构造器可以接受一个`Callable`对象，这就把这二者串起来了，而`FutureTask`本身又是一个`Runnable`，这说明可以把`FutureTask`传递给`Thread`对象的构造器。

```java
public class test5 {
    public static void main(String[] args) throws InterruptedException {
        FutureTask<Integer> result = new FutureTask<Integer>(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                int count = 0;
                for(int i = 1;i <= 100;i++)
                    count += i;
                return count;
            }
        });
        
        new Thread(result).start();
        try {
            System.out.println(result.get());
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

上面例子给出了`FutureTask`的用法，看起来是线程通过`FutureTask`对象间接调用到了`Callable`的call方法。注意，调用`result.get()`时主线程会阻塞一会直到`call`方法执行完毕。

`RunnableFuture`将`Future`和`Runnable`合成一个新接口，没有增加任何一个新方法。

#### Future

```java
public interface Future<V> {
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
    
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    
    boolean isDone();
}
```

`Future`被设计为一个可以异步获得task执行结果的接口。

- `get()`。获得task执行结果，如果task已经执行完，马上返回执行结果；如果task未执行完毕，则阻塞直到执行完毕。
- `get(long timeout, TimeUnit unit)`。上一个函数的超时版本，阻塞直到执行完毕或超时。
- `cancel(boolean mayInterruptIfRunning)`。尝试取消task，返回值代表取消操作成功。
- `isCancelled()`。判断一个task已经被取消了。取消一定是task没有执行完毕前就被取消，也包括根本没有执行就被取消。
- `isDone()`。如果一个任务已经结束, 则返回true。返回true有三种情况：
	- `normal termination`。正常执行完毕。
	- `an exception`。执行过程抛出异常。
	- `cancellation`。task被取消。



