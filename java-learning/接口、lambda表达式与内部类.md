# 接口、lambda表达式与内部类

[TOC]

## 接口

### 接口概念

```java
public interface Comparable{
    int compareTo(Object other);
}
```

接口中的所有方法自动属于public。因此，在接口中声明方法时，不必提供关键字public。

接口可包含多个方法，还可以定义常量，但接口决不能含有实例域，在Java SE 8之后，可以在接口中实现简单方法了。

为了让类实现一个接口，通常有以下步骤：

+ 将类声明为实现给定的接口

  ```java
  class Employee implements Comparable
  ```

+ 对接口中的所有方法进行定义

### 接口的特性

+ 接口不是类，不能使用new运算符实例化一个接口：

  ```java
  x=new Comparable(...);//ERROR
  ```

+ 尽管不能构造接口的对象，却能声明接口的变量：

  ```java
  Comparable x;//OK
  ```

+ 可以使用instanceof检查一个对象是否实现了某个特定的接口：

  ```java
  if(anObject instanceof Comparable){...}
  ```

+ 接口可以被扩展，允许存在多条从具有较高通用性的接口到较高专用性的接口的链：

  ```java
  public interface Moveable{
      void move(double x, double y);
  }
  public interface Powered extends Moveable{
      double milesPerGallon();
  }
  ```

+ 虽然在接口中不能包含实例域或静态方法，但却可以包含常量：

  ```java
  public interface Powered extends Moveable{
      double milesPerGallon();
      double SPEED_LIMIT=95;//a public static final constant
  }
  ```

  与接口中的方法都自动设置为public一样，接口中的域将被自动设为public static final。

+ 有些类只定义了常量，而没有定义方法。如，标准库中一个SwingConstants，其中只包含NORTH等常量。任何实现了SwingConstants接口的类都自动地继承了这些常量，并可以在方法中直接引用NORTH，而不必采用SwingConstants.NORTH这样的繁琐形式。但建议不用，因为违背接口的初衷。

+ 尽管每个类只能够拥有一个超类，但可以实现多个接口：

  ```java
  class Employee implements Cloneable, Comparable
  ```

### 静态方法

在Java SE 8中，允许在接口中增加静态方法。

以往人们通常将静态方法放在伴随类中。在标准库中，你会看到成对的接口和使用工具类，如Path/Paths。

对于Paths类，只包含两个工厂方法。可以由一个字符串序列构造一个文件或目录的路径，如Paths.get("jdk1.8.0", "jre", "bin")。在Java SE 8中，可以为Path接口增加以下方法：

```java
public interface Path{
    public static Path get(String first, String... more){
        return FileSystems.getDefault().getPath(first, more);
    }
}
```

这样一来Paths就不是必要的了。

### 默认方法

可以为接口方法提供一个默认实现。必须用default修饰符标记这样一个方法：

```java
public interface Comparable<T other> {
    default int compareTo(T other){return 0;}
}
```

### 解决默认方法冲突

+ 超类优先。
+ 接口冲突。如果一个接口提供了一个默认方法，另一个接口提供了一个同名而且参数类型（不论是否是默认参数）相同的方法，必须覆盖这个方法来解决冲突。

对于第二点：

```java
interface Named{
    default String getName(){return getClass().getName()+"_"+hashCode();}
}
```

```java
class Student implements Perdon, Named{
    //need to override this method
    public String getName(){return Person.super.getName()}
}
```

## 接口示例

### 接口与回调

回调（callback）是一种常见的程序设计模式。在这种模式下，可以指出某个特定时间发生时应该采取的动作。

```java
public interface ActionListener{
    void actionPerformed(AcctionEvent event);
}
```

```java
public static void main(String[] args){
    ActionListener listener=new TimerPrinter();
    //Constuct a timer that calls the listener
    //once every 10 seconds
    Timer t=new Timer(10000, listener);
    t.start();
}
class TimerPrinter implements ActionListener{
    public void actionPerformed(ActionEvent event){
        System.out.print("At the tone, the time is" + new Date());
        Toolkit.getDefaultTookit().beep();
    }
}
```

```java
javax.swing.Timer 1.0;
Timer(int interval, ActionListener listener);
/*构造一个定时器，每隔interval个毫秒通告listener一次*/
void start();
/*启动定时器，一旦启动成功，定时器将调用监听器的actonPerformed*/
void stop();
/*停止定时器*/
```

### Comparator接口

```java
public interface Comparator<T>{
    int compare(T first, T second);
}
```

```java
class LengthComparator implements Comparator<String>{
    public int compare(String first, String second){
        return first.length()-second.length();
    }
}
//具体完成比较时需要建立一个实例
Comparator<String> comp=new LengthComparator();
if(comp.compare(words[i],words[j])>0) ...;
//对一个数组排序
String[] friends={...};
Arrays.sort(friends, new LengthComparator());
```

### 对象克隆

直接复制一般为浅拷贝。如果原对象和浅克隆对象共享的子对象是不可变的，那么这种共享是安全的。如果子对象是属于一个不可变的类，如String，或者在对象的生命周期中，子对象一直包含不变的常量，没有更改器方法会改变它，也没有方法生成它的引用，也是同样安全的。

注意，Object类中的clone方法声明为protected，所以不能直接调用anObject.clone()。因此，需要：

+ 实现Cloneable接口
+ 重新定义clone方法，并指定public访问修饰符

在这里，Cloneable接口的出现于接口的正常使用没有关系。它没有指定clone方法，这个方法是从Object继承的。这个接口只是作为一个标记。如果一个对象请求克隆，但没有实现这个接口，就会生成一个受查异常。

注意：Cloneable接口是Java提供的一组标记接口之一。标记接口不含任何方法；它的唯一作用就是允许在类型查询中使用instanceof：

```java
if(obj instanceof Cloneable) ...;
```

即使clone的默认（浅拷贝）实现能够满足要求，还是需要实现cloneable接口，将clone重新定义为public，再调用super.clone()。

```java
public Employee implements Cloneable{
    //raise visibility level to public, change return type
    public Employee clone() throws CloneNotSupportedException{
        return (Employee) super.clone();
    }
}
```

对于深拷贝：

```java
public Employee implements Cloneable{
    public Employee clone() throws CloneNotSupportedException{
        Employee cloned=(Employee) super.clone();
        //clone mutable fields
        cloned.hireDay=(Date) hireDay.clone();
        return cloned;
    }
}
```

## lambda表达式

### lambda表达式的语法

+ 如果代码要完成的计算无法放在一个表达式中，就可以写在{}中，并包含显示的return语句。

  ```java
  (String first, String second)->{
      if(first.length()<second.length()) return -1;
      else if(first.length()>second.length()) return 1;
      else return 0;
  }
  ```

+ 即使lambda表达式没有参数，仍然要提供空括号：

  ```java
  ()->{for(int i=0; i<100; i++) System.out.println(i);}
  ```

+ 如果可以推导出一个lambda表达式的参数类型，则可以忽略其类型：

  ```java
  Comparator<String> comp
      =(first, second)
      ->first.length()-second.length();
  ```

+ 如果方法只有一个参数，而且这个参数的类型可以推到得出，那么可以省略小括号：

  ```java
  //instead of (event)->... or (ActionEvent event)->...
  ActionListener listener=event->
      System.out.println("The time is "+new Data());
  
  ```

示例：

```java
String[] planets=new String[]{...};
Arrays.sort(planets, (first, second)->first.length()-second.length());
Timer t=new Timer(1000, event->System.out.println(...));
```

### 函数式接口

对于只有一个抽象方法的接口，需要这种接口的对象时，就可以提供一个lambda表达式。这种接口称为函数式接口。

Comparator是一个函数式接口，可以提供一个lambda表达式：

```java
Arrays.sort(words, (first, second)->first.length()-second.length());
```

lambda表达式可以转换为接口：

```java
Timer t=new Timer(1000, event->{
    System.out,println(...);
});
```

java.util.function包中有一个尤其有用的接口Predicate:

```java
public interface Predicate(T){
    boolean test(T t);
}
```

ArrayList类有一个removeIf方法，有个Predicate参数，这个接口是专门用来传递lambda表达式的。如，将一个数组列表删除所有null值：

```java
list.removeIf(e->e==null);
```

### 方法引用

```java
Timer t=new Timer(1000, event->System.out.println(event));
```

可以直接把println方法传递到Timer构造器：

```java
Timer t=new Timer(1000, System.out::println);
```

表达式System.out::println是一个方法引用（method reference），等价于x->System.out.println(x)。

不考虑字母大小写排序：

```java
Arrays.sort(strings, String::compareToIgnoreCase);
```

综上，要用::操作符分割方法名与对象或类名，有3中情况：

+ object::instanceMethod
+ Class::staticMethod
+ Class::instanceMethod

前两种情况中，类似于x->fun(x)或(x, y)->fun(x, y)。

对于第三种情况，等同于(x, y)->x.fun(y)。

可以在方法引用中使用this参数，如this::equals等同于x->this.equals(x)。super也一样。

```java
class Greeter{
    public void greet(){
        ...;
    }
}
class TimedGreeter extends Greeter{
    public void greet(){
        Timer t=new Timer(1000, super::greet);
        t.start();
    }
}
```

### 构造器引用

构造器引用于方法引用很类似，只不过方法名为new。

假设你有一个字符串列表，可以把它转换为一个Person对象数组，为此要在各个字符串上调用构造器：

```java
ArrayList<String> names=...;
Stream<Person> stream=names.stream().map(Person::new);
List<Person> people=stream.collect(Collectors.toList());
```

可以用数组类型建立构造器引用。如，int[]::new是一个构造器引用，有一个参数，即数组长度，等价于x->new int[x]。

最后将Person[]::new传入toArray方法：

```java
Person[] people=stream.toArray(Person[]::new);
```

### 变量作用域

+ 在lambda表达式中，只能引用值不会改变的变量。下面做法不合法：

  ```java
  public static void countDown(int start, int delay){
      ActionListener listener=event->{
          start--;//Error
          ...;
      }
      ...;
  }
  ```
  
+ 如果在lambda表达式中引用变量，而这个变量可能在外部改变，这也是不合法的：

  ```java
  for(int i=0; i<100; i++){
      ActionListener listener=event->{
          System.out.println(i);//Error
      }
  }
  ```

+ 在lambda表达式中声明与一个局部变量同名的参数或局部变量是不合法的：

  ```java
  Path first=Paths.get(...);
  Comparator<String> comp=
      (first, second)->first.length()-second.length();
  //Error
  ```

+ 在一个lambda表达式中使用this关键字时，是指创建这个lambda表达式的方法的this参数：

  ```java
  public class Application(){
      public void init(){
          ActionListener listener=event->
          {
              System.out.println(this.toString());
              ...;
          }
      }
  }
  ```

### 处理lambda表达式

使用lambda表达式的重点在于延迟执行，如果需要立即执行，就无需包装在lambda表达式中。

用到的场景如下：

+ 在一个单独的线程中运行代码
+ 多次运行代码
+ 在算法的适当位置运行代码，如Comparator接口
+ 发生在某种情况时执行代码
+ 只有在必要时才运行代码

常用的函数接口如下：

| 函数式接口 | 参数类型 | 返回类型 | 抽象方法名 | 描述 | 其他方法 |
| ---------- | -------- | -------- | ---------- | ---- | -------- |
|Runnable|无|void|run|作为无参数或返回值的动作运行||
|Supplier<T>|无|T|get|提供一个T类型的值||
|Consumer<T>|T|void|accept|处理一个T类型的值|andThen|
|BiConsumer<T, U>|T, U|void|accept|处理一个T和U类型的值|andThen|
|Function<T, R>|T|R|apply|有一个T类型参数的函数|compose,andThen, identity|
|BiFunction<T, U, R>|T, U|R|apply|有T和U类型参数的函数|andThen|
|UnaryOperator<T>|T|T|apply|类型T上的一元操作符|compose, andThen, dentity|
|BinaryOperator<T>|T, T|T|apply|类型T上的二元操作符|andThen, maxBy, minBy|
|Predicate<T>|T|boolean|test|布尔值函数|and, or, negate, isEqual|
|BiPredicate<T, U>|T, U|boolean|test|有两个参数的布尔值函数|and, or, negate|

如果想要重复一个动作n次，则可以选择Runnable接口：

```java
public static void repeat(int n, Runnable action){
    for(int i=0; i<n; i++) action.run();
}
repeat(100, ()->System.out.println(...));
```

如果希望告诉这个动作出现在哪一次迭代中，则需要构造合适的接口：

```java
public interface IntConsumer{
    void accept(int value);
}
public static void repeat(int n, IntConsumer action){
    for(int i=0; i<n; i++) action.accept(i);
}
repeat(100, i->System.out.println(i));
```

以下列出了基本类型的规范，最好使用这些特殊化规范来减少自动装箱。出于这个原因，上面我们用的是IntConsumer而不是Consumer<Integer>。

| 函数式接口 | 参数类型 | 返回类型 | 抽象方法名 |
| ---------- | -------- | -------- | ---------- |
|BooleanSupplier|none|boolean|getAsBoolean|
|PSupplier|none|p|getAsP|
|PConsumer|p|void|accept|
|ObjPComsumer<T>|T, p|void|accept|
|PFunction<T>|p|T|apply|
|PToQFunction<T>|p|q|applyAsQ|
|ToPFunction<T>|T|p|applyAsP|
|ToPBiFunction<T, U>|T, U|p|applyAsP|
|PUnaryOperator|p|p|applyAsP|
|PBinaryOperator|p, p|p|appluAsP|
|PPredicate|p|boolean|test|

> 注：p, q为int, long, double; P, Q为Int, Long, Double。

> 注：大多数标准函数式接口都提供了非抽象方法来生成或合并函数。如，Predicate.isEqual(a)等同于a::equals。已经提供了默认方法and、or或negate来合并谓词。如，Predicate.isEqual(a).or(Predicate.isEqual(b))就等同于  x->a.equals(x) || b.equals(x)。

> 注：如果设计接口，其中只有一个抽象方法，可以用@FunctionalInterface注解来标记接口。有两个优点：若无意中增加了另一个非抽象方法，编译器会产生一个错误信息。另外javadoc页里会指出你的接口是一个函数式接口。

### 再谈Comparator

Comparator接口包含很多方便的静态方法来创建比较器。这些方法可以用于lambda表达式或方法引用。

静态comparing方法取一个“键提取器”函数，它将类型T映射为一个可比较的类型（如String）。对要比较的对象应用这个函数，然后对返回的键完成比较。如，假设有一个Person对象数组，可如下按名字对这些对象排序：

```java
Arrays.sort(people, Comparator.comparing(Person::getName));
```

可以把比较器与thenComparing方法串起来：

```java
Arrays.sort(people, 
          Comparator.comparing(Person::getLastName).thenComparing(Person::getFirstName));
```

这些方法有很多变体形式。可以为comparing和thenComparing方法提取的键指定一个比较器。如，可以如下根据人名长度完成排序：

```java
Arrays.sort(people, Comparator.comparing(Person::getName,
                                         (s,t)->Integer.compare(s.length, t.lenght())));
```

另外还有变体形式避免装箱：

```java
Arrays.sort(people, Comparator.comparingInt(p->p.getName().length()));
```

如果键函数可以返回null，可能就要用到nullsFirst和nullsLast适配器。这些静态方法会修改现有的比较器，从而在遇到null值时不会抛出异常，而是将这个值标记为小于或大于正常值。

```java
Arrays.sort(people,
            Comparator.comparing(Person::getMiddleName(),
                                 Comparator.nullsFirst(naturalOrder())));
```

静态reverseOrder方法会提供自然顺序的逆序。要让比较器逆序比较，可以使用reversed实例方法。如naturalOrder().reversed()等同于reverseOrder()。

## 内部类

### 使用内部类访问对象状态

一个外围类含有内部类，这并不意味着每个外围类都有对应的内部类的实例。

内部类既可以访问自身的数据域，也可以访问创建它的外围类对象的数据域。

### 内部类的特殊语法机制

使用外围类引用的语法：

```java
//在内部类中对外围类的变量的引用
OuterClass.this.value
```

反过来，可以采用下列语法格式更加明确地编写内部对象的构造器：

```java
outerObject.new InnerClass(construction parameters)
```

可以通过显示地命名将外围类引用设置为其他的对象。假设TimePrinter是TalkingClock的公有内部类：

```java
TalkingClock jabberer=new TalkingClock(1000, true);
TalkingClock.TimePrinter listener=jabberer.new TimePrinter();
```

需要注意，在外围类的作用域之外，可以这样引用内部类：

```java
OuterClass.InnerClass
```

注：内部类中声明的所有静态域都必须是final的。

### 局部内部类

```java
public void start(){
    class TimePrinter implements ActionListener{
        public void actionPerformed(ActionEvent event){
            System.out.println(...);
            ...;
        }
    }
    ActionListener listener=new TimePrinter();
    Timer t=new Timer(interval, listener);
    t.start();
}
```

局部类不能用public或private访问说明符进行声明。它的作用域被限定在声明这个局部类的块中。

局部类有一个优势，即对外部世界完全隐藏。

### 由外部方法访问变量

局部类还有一个优点：它们不仅能够访问包含它们的外部类，还可以访问局部变量。不过那些局部变量必须为final的。

```java
public void start(int interval, boolean beep){
    class TimePrinter implements ActionListener{
        public void actionPerformed(ActionEvent event){
            if(beep) ...;
        }
    }
    ActionListener listener=new TimePrinter();
    Timer t=new Timer(interval, listener);
    t.start();
}
```

控制流程如下：

+ 调用start方法。
+ 调用TimePrinter构造器，初始化对象变量listener。
+ 将listener引用传递给Timer构造器，定时器开始计时，start方法结束。此时，start方法的beep参数变量不复存在。
+ 然后内部类访问beep...

为了能够让actionPerformed方法工作，TimePrint类在beep域释放之前将beep域用start方法的局部变量进行备份。

有时，final限制显得并不方便。如，假设向更新在一个封闭作用域内的计数器。这里想要统计一下在排序过程中调用compareTo方法的次数：

```java
int counter=0;
Date[] dates=new Date[100];
for(int i=0; i<dates.length; i++){
    dates[i]=new Date(){
        public int compareTo(Date other){
            counter++;//ERROR
            return super.compareTo(other);
        }
    }
}
Arrays.sort(dates);
```

由于清楚地知道counter需要更新，所以不能将counter声明为final。由于Integer对象不可变，所以也不能用Integer替代它。但可以用长度为1的数组：

```java
int[] counter=new int[1];
for(int i=0; i<dates.length; i++){
    dates[i]=new Date(){
        public int compareTo(Date other){
            counter[0]++;
            return super.compareTo(other);
        }
    }
}
```

### 匿名内部类

下面的技巧称为“双括号初始化”，这里利用了内部类的语法。假设你想构造一个数组列表，并将它传递到一个方法：

```java
ArrayList<String> friends=new ArrayList<>();
friends.add(...);
invite(friends);
```

如果不再需要这个数组列表，最好让它作为一个匿名列表。不过作为一个匿名列表，该如何为它添加元素呢？

```java
invite(new ArrayList<String>(){{add("...");add("...");}})
```

注意这里的双括号。外层括号建立了ArrayList的一个匿名子类。内层括号则是一个对象构造块。

> 注意：以下语句对匿名子类做测试会失败：

```java
if(getClass()!=other.getClass()) return false;
```

> 提示：生成日志或调试信息时，通常希望包含当前类的类名，如：

```java
System.err.println("Something awful happened in"+getClass());
```

不过这对于静态方法不奏效。毕竟调用getClass时调用的是this.getClass()，而静态方法没有this。所以应该使用以下表达式：

```java
/*new Object(){}会建立Object的一个匿名子类的一个匿名对象，getEnclosingClass得到其外围类*/
new Object(){}.getClass().getEnclosingClass();
```

### 静态内部类

有时候，使用内部类只是为了把一个类隐藏在另外一个类的内部，并不需要内部类引用外围类对象。为此可以将内部类声明为static，以便取消产生的引用。

```java
class ArrayAlg{
    public static class Pair{
        ...;
    }
    public static Pair minmax(double[] values){
        ...;
        return new Pair(min, max);
    }
}
```

在大型项目中，除了定义包含一对字符串的Pair类之外，其他程序员也很可能使用这个名字，这样就会产生名字冲突。用静态内部类的方式可以解决该问题：

```java
ArrayAlg.Pair p=ArrayAlg.minmax(d);
```

> 注释：在内部类不需要访问外围类对象的时候，应该使用静态内部类。

> 注释：与常规内部类不同，静态内部类可以有静态域和方法。

> 注释：声明在接口中的内部类自动成为static和public类。

## 代理

利用代理可以在运行时创建一个实现了一组给定接口的新类。这种功能只有在编译时无法确定需要实现哪个接口时才有必要使用。

### 创建代理对象

要想创建一个代理对象，需要使用Proxy类的newProxyInstance方法。有三个参数：

+ 类加载器：作为Java安全模型的一部分，对于系统类和从因特网上下载下来的类，可以使用不同的类加载器。目前，用null表示使用默认的类加载器。
+ Class对象数组，每个元素都是需要实现的接口。
+ 调用处理器。

使用代理有以下原因：

+ 路由对远程服务器的方法调用。
+ 在程序运行期间，将用户接口事件与动作关联起来。
+ 为调试，跟踪方法调用。

在以下程序中，使用代理对象对二分查找进行跟踪：

```java
public class ProxyTest{
    public static void main(String[] args){
        Object[] elements=new Object[1000];
        //fill elements with proxies for the integers 1 ... 1000
        for(int i=0; i<elements.length; i++){
            Integer value=i+1;
            InvocationHandler handler=new TraceHandler(value);
            Object proxy=Proxy.newProxyInstance(null, new Class[]{Comparable.class}, handler);
            elements[i]=proxy;
        }
        Integer key=new Random().nextInt(elements.length)+1;
        int result=Arrays.binarySearch(elements, key);
        if(result>=0) System.out.println(elements[result]);
    }
}
class TraceHandler{
    private Object target;
    public TraceHandler(Object t){
        target=t;
    }
    public Object invoke(Object proxy, Method m, Obejct[] args) throws Throwable{
        System.out.println(target);
        System.out.println("."+m.getName()+"(");
        if(args!=null){
            for(int i=0; i<args.length; i++){
                System.out.print(args[i]);
                if(i<args.length-1) System.out.print(", ");
            }
        }
        System.out.println(")");
        //invoke actual method
        return m.invoke(target, args);
    }
}
```

由于数组中填充了代理对象，所以compareTo调用了TraceHandler类中的invoke方法。这个方法打印了方法名和参数，之后用包装好的Integer对象调用compareTo。

输出结果示例：

```java
500.compareTo(288)
250.compareTo(288)
    ...
288.toString()
```

> 注意：即使不属于Comparable接口，toString方法也被代理，而且有相当一部分的Object方法都被代理。

### 代理类特性

+ 代理类是在程序运行过程中创建的，一旦被创建，就变成了常规类，与虚拟机中的任何其他类没区别。
+ 所有的代理类都扩展与Proxy类。一个代理类只有一个实例域——调用处理器，它定义在Proxy的超类中。
+ 所有的代理类都覆盖了Object类中的方法toString、equals和hashCode。这些方法仅仅调用了调用处理器的invoke。Object类中的其他方法（如clone和getClass）没有被重新定义。
+ 代理类一定是public和final。如果代理类实现的所有接口都是public的，代理类就不属于某个特定的包；否则，所有非公有的接口都必须属于同一个包，同时，代理类也属于这个包。
+ 可以通过调用Proxy类中的isProxyClass方法检测一个特定的class对象是否代表一个代理类。

```java
java.lang.reflect.invocationHandler 1.3;
Object invoke(Object proxy, Method method, Object[] args);
/*定义了代理对象调用方法时希望执行的动作*/
```

```java
java.lang.reflect.Proxy 1.3;
static Class<?> getProxyClass(ClassLoader loader, Class<?>... interfaces);
/*返回实现指定接口的代理类*/
static Object newProxyInstance(ClassLoaderloader, Class<?>[] interfaces, InvocationHandler handler);
/*构造实现指定接口的代理类的一个新实例。所有方法会调用给定处理器对象的invoke方法*/
static boolean isProxyClass(Class<?> cl);
/*如果cl是一个代理类则返回true*/
```

