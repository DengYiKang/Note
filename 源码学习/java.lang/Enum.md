# Enum

[TOC]

## 一个简单的枚举类实例

```java
public enum Fruit{
    APPLE(1),ORANGE(2),BANANA(3);
    int code;

    Fruit(int code){
        this.code=code;
    }
}
```

## 反编译文件

```java
public final class Fruit extends Enum<Fruit>
{

    //返回所有类对象
    public static Fruit[] values()
    {
        return (Fruit[])$VALUES.clone();
    }

    //根据name参数返回类对象
    public static Fruit valueOf(String s)
    {
        return (Fruit)Enum.valueOf(Fruit, s);
    }

    //s是name参数，i是ordinal参数，j是自定义参数
    private Fruit(String s, int i, int j)
    {
        super(s, i);
        code = j;
    }

    //可以看出，用户定义的所有的枚举常量都作为该类的类变量
    public static final Fruit APPLE;
    public static final Fruit ORANGE;
    public static final Fruit BANANA;
    //自定义的成员变量
    int code;
    //枚举常量数组，保存所有该枚举类型对应的枚举常量
    private static final Fruit $VALUES[];

    //初始化
    static
    {
        APPLE = new Fruit("APPLE", 0, 1);
        ORANGE = new Fruit("ORANGE", 1, 2);
        BANANA = new Fruit("BANANA", 2, 3);
        $VALUES = (new Fruit[] {
            APPLE, ORANGE, BANANA
        });
    }
}
```

编译器会自动处理枚举：

+ 将枚举类型继承Enum类
+ 将所有定义的枚举常量定义为该类的类变量
+ 定义枚举常量数组，保存所有该枚举类型对应的枚举常量
+ 在static块中对所有枚举常量进行初始化
+ 生成values和valueOf方法，values方法用于返回所有的枚举常量，valueOf返回给定name对应的枚举常量

## Enum抽象类源代码

```java
public abstract class Enum<E extends Enum<E>>  
        implements Comparable<E>, Serializable {  
      
    private final String name;  
  
    // 当前枚举常量名称  
    public final String name() {  
    	return name;  
    }  
  
    private final int ordinal;  
  
    // 当前枚举常量次序，从0开始  
    public final int ordinal() {  
    	return ordinal;  
    }  
  
    // 专有构造器，我们无法调用。该构造方法用于由响应枚举类型声明的编译器发出的代码。   
    protected Enum(String name, int ordinal) {  
        this.name = name;  
        this.ordinal = ordinal;  
    }  
  
    // 返回枚举常量的名称，默认是返回name值。可以重写该方法，输出更加友好的描述。  
    public String toString() {  
    	return name;  
    }  
  
    // 比较当前枚举常量是否和指定的对象相等。因为枚举常量是单例的，所以直接调用==操作符。子类不可以重写该方法。  
    public final boolean equals(Object other) {   
        return this==other;  
    }  
  
    // 返回该枚举常量的哈希码。和equals一致，该方法不可以被重写。  
    public final int hashCode() {  
        return super.hashCode();  
    }  
  
    // 因为枚举常量是单例的，所以不允许克隆。  
    protected final Object clone() throws CloneNotSupportedException {  
    	throw new CloneNotSupportedException();  
    }  
  
    // 比较该枚举常量和指定对象的大小。它们的类型要相同，根据它们在枚举声明中的先后顺序来返回大小（前面的小，后面的大）。子类不可以重写该方法  
    public final int compareTo(E o) {  
        Enum other = (Enum)o;  
        Enum self = this;  
        if (self.getClass() != other.getClass() && // optimization  
                self.getDeclaringClass() != other.getDeclaringClass())  
            throw new ClassCastException();  
        return self.ordinal - other.ordinal;  
    }  
  
    // 得到枚举常量所属枚举类型的Class对象  
    public final Class<E> getDeclaringClass() {  
        Class clazz = getClass();  
        Class zuper = clazz.getSuperclass();  
        return (zuper == Enum.class) ? clazz : zuper;  
    }  
  
    // 返回带指定名称的指定枚举类型的枚举常量。名称必须与在此类型中声明枚举常量所用的标识符完全匹配。不允许使用额外的空白字符。  
    public static <T extends Enum<T>> T valueOf(Class<T> enumType,  
                                                String name) {  
        T result = enumType.enumConstantDirectory().get(name);  
        if (result != null)  
            return result;  
        if (name == null)  
            throw new NullPointerException("Name is null");  
        throw new IllegalArgumentException(  
            "No enum const " + enumType +"." + name);  
    }  
  
    // 不允许反序列化枚举对象  
    private void readObject(ObjectInputStream in) throws IOException,  
        ClassNotFoundException {  
            throw new InvalidObjectException("can't deserialize enum");  
    }  
  
    // 不允许反序列化枚举对象  
    private void readObjectNoData() throws ObjectStreamException {  
        throw new InvalidObjectException("can't deserialize enum");  
    }  
  
    // 枚举类不可以有finalize方法，子类不可以重写该方法  
    protected final void finalize() { }  
}  
```

注意getDeclaringClass方法：

```java
// 得到枚举常量所属枚举类型的Class对象  
public final Class<E> getDeclaringClass() {  
    Class clazz = getClass();  
    Class zuper = clazz.getSuperclass();  
    return (zuper == Enum.class) ? clazz : zuper;  
}  
```

为什么不直接返回getClass方法呢？考虑以下代码：

```java
public enum EnumTest {
    APPLE(1) {
        String getName() {
            return "123";
        }
    }, ORANGE(2), BANANA(3);
    int code;

    EnumTest(int code) {
        this.code = code;
    }
}

```

经过编译后，APPLE就成了EnumTest的内部类，枚举常量APPLE的类名为EnumTest\$1，而其他枚举常量的类名为EnumTest。对于APPLE的getClass方法，返回的是EnumTest\$1，而APPLE的getClass().getSuperclass()方法返回的是EnumTest。

## 常见的面试题

#### 1、枚举允许继承类吗？

枚举不允许继承类。Jvm在生成枚举时已经继承了Enum类，由于Java语言是单继承，不支持再继承额外的类（唯一的继承名额被Jvm用了）。

#### 2、枚举允许实现接口吗？

枚举允许实现接口。因为枚举本身就是一个类，类是可以实现多个接口的。

#### 3、枚举可以用等号比较吗？

枚举可以用等号比较。Jvm会为每个枚举实例对应生成一个类对象，这个类对象是用public static final修饰的，在static代码块中初始化，是一个单例。

#### 4、可以继承枚举吗？

 不可以继承枚举。因为Jvm在生成枚举类时，将它声明为final。

#### 5、枚举是单例吗？

是单例的，枚举是实现单例模式的一种方式。

#### 6、当使用compareTo()比较枚举时，比较的是什么？

枚举类型的compareTo()方法比较的是枚举类对象的ordinal的值。

#### 7、当使用equals()比较枚举的时候，比较的是什么？

枚举类型的equals()方法比较的是枚举类对象的内存地址，作用与等号等价。