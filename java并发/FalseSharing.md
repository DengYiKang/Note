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

不考虑object header等字节空间(实际上object header占12字节，this变量占4字节，总共16字节)，其内部变量有8个long类型的变量，恰好为64字节。在64字节缓存行的体系里面，这样能保证`array`数组的相邻元素占用不同的缓存行。

可以用jol工具查看object的布局：

```shell
FalseSharing$SomePopularObject object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4        (object header)                           43 c1 00 20 (01000011 11000001 00000000 00100000) (536920387)
     12     4        (alignment/padding gap)                  
     16     8   long SomePopularObject.value                   0
     24     8   long SomePopularObject.p1                      0
     32     8   long SomePopularObject.p2                      0
     40     8   long SomePopularObject.p3                      0
     48     8   long SomePopularObject.p4                      0
     56     8   long SomePopularObject.p5                      0
     64     8   long SomePopularObject.p6                      0
     72     8   long SomePopularObject.p7                      1
Instance size: 80 bytes
Space losses: 4 bytes internal + 0 bytes external = 4 bytes total
```

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

```java
public class SomePopularObject {
    @Contended
    public volatile long usefulVal;
    public volatile long anotherVal;
}
```

### JMH测试

```java
package benchmark;

import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;
import sun.misc.Contended;


@State(Scope.Benchmark)
//注意@Contented应该与-XX:-RestrictContended配合使用
//-XX:ContendedPaddingWidth允许你控制padding大小，默认是128
@Fork(value = 1, jvmArgsPrepend = "-XX:-RestrictContended")
@Warmup(iterations = 3)
@Measurement(iterations = 10)
public class ContentedBenchmarks {
    private UnpaddedObject unpaddedObject = new UnpaddedObject();
    private PaddedObject paddedObject = new PaddedObject();


    @Group("unpadded")
    @GroupThreads(10)
    @Benchmark
    public long updateUnpaddedA() {
        return unpaddedObject.a++;
    }

    @Group("unpadded")
    @GroupThreads(10)
    @Benchmark
    public long updateUnpaddedB() {
        return unpaddedObject.b++;
    }

    @Group("padded")
    @GroupThreads(10)
    @Benchmark
    public long updatePaddedA() {
        return paddedObject.a++;
    }

    @Group("padded")
    @GroupThreads(10)
    @Benchmark
    public long updatePaddedB() {
        return paddedObject.b++;
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(ContentedBenchmarks.class.getSimpleName())
                .build();
        new Runner(opt).run();
    }

    static class UnpaddedObject {
        public long a;
        public long b;
    }

    static class PaddedObject {
        @Contended
        public long a;
        public long b;
    }
}

```

结果：

```
Benchmark                                      Mode  Cnt          Score         Error Units
ContentedBenchmarks.padded                    thrpt   10  284236715.113 ± 7925784.208  ops/s
ContentedBenchmarks.padded:updatePaddedA      thrpt   10  141359994.411 ± 3961123.177  ops/s
ContentedBenchmarks.padded:updatePaddedB      thrpt   10  142876720.702 ± 5500894.467  ops/s
ContentedBenchmarks.unpadded                  thrpt   10  158332107.536 ± 1287695.576  ops/s
ContentedBenchmarks.unpadded:updateUnpaddedA  thrpt   10   82114654.267 ± 2191121.210  ops/s
ContentedBenchmarks.unpadded:updateUnpaddedB  thrpt   10   76217453.269 ± 1938877.529  ops/s
```



