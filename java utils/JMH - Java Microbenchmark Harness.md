# JMH - Java Microbenchmark Harness

## 配置

在maven项目的pom文件中添加依赖：

```xml
<dependency>
	<groupId>org.openjdk.jmh</groupId>
	<artifactId>jmh-core</artifactId>
	<version>1.27</version>
</dependency>
<dependency>
	<groupId>org.openjdk.jmh</groupId>
	<artifactId>jmh-generator-annprocess</artifactId>
	<version>1.27</version>
</dependency>
<dependency>
	<groupId>org.openjdk.jol</groupId>
	<artifactId>jol-core</artifactId>
	<version>0.14</version>
</dependency>
```

## JMH Benchmark Modes

JMH提供了以下mode作为测试模式，它们都是枚举`Mode`的属性：

+ `Throughout`：吞吐量。这是默认的模式，单位为`ops/s`，即`operations per second`。
+ `Average Time`：平均时长。
+ `Sample Time`：采样模式，列出执行时长，包括`min_time`和`max_time`（因为有多个`iterations`）。
+ `Single Shot Time`：方法的单次调用时间/一次批处理的总调用时间。
+ `All`：包含上述所有模式。

## Benchmark Time Units

JMH允许指定时间单位，以下参数为枚举`TimeUnit`的属性，作为注解`@OutputTimeUnit`的值传入：

+ `NANOSECONDS`：纳秒， $10^{−9}$ seconds.
+ `MICROSECONDS`：微妙，$10^{-6}$ seconds.
+ `MILLISECONDS`：毫秒，$10^{-3}$ seconds.
+ `SECONDS`
+ `MINUTES`
+ `HOURS`
+ `DAYS`

## Benchmark State

Benchmark State一般用于benchmark之间共享状态。

State Class一般作为参数注入到benchmark测试函数中。

```java
public class MyBenchmark {

    //MyState为Group内共享
    @State(Scope.Group)
    public static class MyState {
        public int a = 1;
        public int b = 2;
        public int sum ;
    }

    /**
     * testMethod1, testMethod2同属group one， 共享同一个MyState
     * testMethod3, testMethod4同属group two， 共享同一个Mystate
     * group one与group two不共享Mystate
     */ 
    @Benchmark
    @Group("one")
    public void testMethod1(MyState state) {
        state.sum = state.a + state.b;
    }
    
    @Benchmark
    @Group("one")
    public void testMethod2(MyState state){
        state.sum++;
    }
    
	@Benchmark
    @Group("two")
    public void testMethod3(MyState state) {
        state.sum = state.a + state.b;
    }
    
    @Benchmark
    @Group("two")
    public void testMethod4(MyState state){
        state.sum++;
    }

}
```

### State Scope

以下属于枚举`Scope`的属性：

+ `Thread`：一个线程共享一个state（最小的scope）。
+ `Benchmark`：单个benchmark的所有线程共享一个state。
+ `Group`：组内共享，即同组的所有benchmark共享一个state。

### State Class Requirements

作为state的类需要满足以下条件：

+ 这个类必须被声明为`public`的。
+ 如果这个类是嵌套类，那么必须被声明为`static`的。
+ 这个类必须要有一个共有的无参构造函数。

### State Object @Setup and @TearDown

`@Setup`和`@TearDown`分别用于注解用于初始化的方法和用于清理的方法。

例如：

```java
public class MyBenchmark {

    @State(Scope.Thread)
    public static class MyState {

        @Setup(Level.Trial)
        public void doSetup() {
            sum = 0;
            System.out.println("Do Setup");
        }

        @TearDown(Level.Trial)
        public void doTearDown() {
            System.out.println("Do TearDown");
        }

        public int a = 1;
        public int b = 2;
        public int sum ;
    }

    @Benchmark
    public void testMethod(MyState state) {
        state.sum = state.a + state.b;
    }
}
```

可以对`@Setup`和`@TearDown`设置以下值用于控制粒度：

+ `Level.Trial`：一次执行对应一个benchmark完整的运行，包括所有的`warmup`和所有的`benchmark iteration`。
+ `Level.Iteration`：一次执行对应一个`Iteration`。
+ `Level.Invocation`：一次执行对应benchmark测试方法的一次调用。

> 注意，一个benchmark有多轮iterations，一个iteration有多个invocations

 