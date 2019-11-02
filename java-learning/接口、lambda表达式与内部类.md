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



