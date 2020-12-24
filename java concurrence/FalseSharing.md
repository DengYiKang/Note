# FalseSharing

cpu使用缓存cache来降低访问主存的消耗，模式类似下图：

![Image title](../pic/3.png)

在缓存系统中，内存是以缓存行(cache lines)的单元存储的。缓存行一般以$2^n$个连续的字节组成，典型的有32~256字节。最常见的一行的大小是64字节。FalseSharing是指，在并发的环境下，多个线程对处于同一缓存行中的变量进行写操作而导致性能的下降。当一个线程对某个缓存行进行写操作时，为了保持缓存的一致性，在其他线程所持有的同一个缓存行将被设置为无效的，需要重新进行读取操作。因此，如果多个线程访问相邻的变量，FalseSharing将会导致性能的下降。

在Java7及之前，通常在相邻的变量之间填充padding。例如：

```java
public class SomePopularObject{
    public volatile long usefulVal;
    public volatile long t1, t2, t3, t4, t5, t6, t7;
}
SomePopularObject[] array=new SomePopularObject[n];
```

不考虑class head等字节空间，其内部变量有8个long类型的变量，恰好为64字节。在64字节缓存行的体系里面，这样能保证`array`数组的相邻元素占用不同的缓存行。

但JVM会对dead code进行优化，这些dead code将会被编译器删除，那么我们可以这样做来避免被优化掉：

```java
public class SomePopularObject{
    public volatile long usefulVal;
    public volatile long t1, t2, t3, t4, t5, t6, t7=1L;
    public long preventOptmisation(){
        return t1+t2+t3+t4+t5+t6+t7;
    }
}
SomePopularObject[] array=new SomePopularObject[n];
```

所幸的是，Java8引入了`@Contented`注解来避免手动添加dead variables。JVM将在被`@Contented`注解的域的后面插入128字节的padding（为什么是128字节而不是64字节？我看有些资料解释prefetcher指令取得是2块缓存行，所以是128字节，然而还是不理解）。