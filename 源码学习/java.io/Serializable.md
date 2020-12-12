# Serializable

```java
public interface Serializable {
}
```

该接口没有方法没有域，仅仅作为一个标示作用，表示是否能序列化和反序列化。

当父类没有继承序列化接口时，子类如果需要序列化，那么必须保证以下几点：

+ 子类必须负责保存和恢复父类的可访问(`public`, `protect`, `package`)的状态(fields)
+ 父类必须有可访问的无参构造器（为了初始化父类的状态，用于反序列化阶段）

如果需要自定义序列化和反序列化，可以实现以下方法：

```java
private void writeObject(java.io.ObjectOutputStream out) throws IOException；
private void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException;
private void readObjectNoData() throws ObjectStreamException;
```

序列化的过程用`serialVersionUID`来关联每个序列化类，用于验证发送方和接收方是否为这类加载了对应的序列化对象，如果接收方检测到`serialVersionUID`与发送方的不同时，将会抛出`InvalidClassException`。

java doc建议最好在序列化的类中声明`serialVersionUID`，因为其默认的计算方式与编译器的实现有关，因此可能会导致`InvalidClassException`异常。因此为了保证在不同的编译器之间传输，建议声明`serialVersionUID`。同时也被建议用`private`修饰。

注意，数组对象无法显示地声明`serialVersionUID`。

其他：

+ 序列化时，只对对象的状态进行保存，而不管对象的方法；
+ 当一个父类实现序列化，子类自动实现序列化，不需要显式实现Serializable接口；
+ 当一个对象的实例变量引用其他对象，序列化该对象时也把引用对象进行序列化；
+ 并非所有的对象都可以序列化，至于为什么不可以，有很多原因了,比如：