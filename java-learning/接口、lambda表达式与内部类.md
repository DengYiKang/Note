# 接口、lambda表达式与内部类

## 接口

### 接口概念

```java
public interface Comparable{
    int compareTo(Object other);
}
```

接口中的所有方法自动属于public。因此，在接口中声明方法时，不必提供关键字public。

接口可包含多个方法，还可以定义常量，==但接口决不能含有实例域==，在Java SE 8之后，可以在接口中实现简单方法了。

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

  ==与接口中的方法都自动设置为public一样，接口中的域将被自动设为public static final==。

+ 有些类只定义了常量，而没有定义方法。如，标准库中一个SwingConstants，其中只包含NORTH等常量。任何实现了SwingConstants接口的类都自动地继承了这些常量，并可以在方法中==直接引用NORTH==，而不必采用SwingConstants.NORTH这样的繁琐形式。但建议不用，因为违背接口的初衷。

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

==注意==，Object类中的clone方法声明为protected，所以不能直接调用anObject.clone()。因此，需要：

+ 实现Cloneable接口
+ 重新定义clone方法，并指定public访问修饰符

在这里，Cloneable接口的出现于接口的正常使用没有关系。它没有指定clone方法，这个方法是从Object继承的。这个接口只是作为一个标记。如果一个对象请求克隆，但没有实现这个接口，就会生成一个受查异常。

==注意==：Cloneable接口是Java提供的一组标记接口之一。标记接口不含任何方法；它的唯一作用就是允许在类型查询中使用instanceof：

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

+ 在一个lambda表达式中使用this关键字时，是指==创建这个lambda表达式的方法==的this参数：

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