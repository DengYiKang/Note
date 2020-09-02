# Collection, Map常用API

[TOC]

![img](pic/pic1)

## List

### List常用方法

```java
public interface List<E>{
    Iterator iterator();
    int size();
    void add(int index, E element);
	boolean addAll(Collection<? extends E> c);
	void clear();
    
	E remove(int index);
	boolean remove(Object o);
	boolean removeAll(Collection<?> c);
    
	E set(int index, E element);
	E get(int index);
    
	boolean isEmpty();
	boolean contains(Object o);
	boolean containsAll(Collection<?> c);
	Object[] toArray();
}

```

### ArrayList基本操作

```java
public class ArrayListTest {
    public static void main(String[] agrs){
        //创建ArrayList集合：
        List<String> list = new ArrayList<String>();
        
        //添加功能：
        list.add("...");
        
        //修改功能：
        list.set(0,"...");
        
        //获取功能：
        String element = list.get(0);
        System.out.println(element);
        
        //迭代器遍历集合：(ArrayList实际的迭代器是Itr对象)
        Iterator<String> iterator =  list.iterator();
        while(iterator.hasNext()){
            String next = iterator.next();
            System.out.println(next);
        }
        
        //for循环迭代集合：
        for(String str:list){
            ...;
        }
        
        //判断功能：
        boolean isEmpty = list.isEmpty();
        boolean isContain = list.contains("...");
        
        //长度功能：
        int size = list.size();
        
        //把集合转换成数组：
        String[] strArray = list.toArray(new String[]{});
        
        //删除功能：
        list.remove(0);
        list.remove("...");
        list.clear();
    }
}
```

### LinkedList基本操作

```java
public class LinkedListTest {
    public static void main(String[] agrs){
        List<String> linkedList = new LinkedList<String>();
        
        //添加功能：
        linkedList.add("...");
        
        //修改功能:
        linkedList.set(0,"...");
        
        //获取功能：
        String element = linkedList.get(0);
        
        //遍历集合：
        Iterator<String> iterator =  linkedList.iterator();
        while(iterator.hasNext()){
            String next = iterator.next();
            System.out.println(next);
        }
        
        //for循环迭代集合：
        for(String str:linkedList){
            ...;
        }
        
        //判断功能：
        boolean isEmpty = linkedList.isEmpty();
        boolean isContains = linkedList.contains("...");
        
        //长度功能：
        int size = linkedList.size();
        
        //删除功能：
        linkedList.remove(0);
        linkedList.remove("...");
        linkedList.clear();
        
        //队列功能。LinkedList类实现了Queue接口，因此我们可以把LinkedList当成Queue来用。
        //add()和remove()方法在失败的时候会抛出异常(不推荐)
        Queue<String> queue = new LinkedList<String>();
        //添加元素
        queue.offer("a");
        queue.offer("b");
        System.out.println("poll="+queue.poll()); //返回第一个元素，并在队列中删除
        System.out.println("element="+queue.element()); //返回第一个元素
        System.out.println("peek="+queue.peek()); //返回第一个元素 
    }
}
```

**offer，add 区别：**

一些队列有大小限制，因此如果想在一个满的队列中加入一个新项，多出的项就会被拒绝。

这时新的 offer 方法就可以起作用了。它不是对调用 add() 方法抛出一个 unchecked 异常，而只是得到由 offer() 返回的 false。

**poll，remove 区别：**

remove() 和 poll() 方法都是从队列中删除第一个元素。remove() 的行为与 Collection 接口的版本相似， 但是新的 poll() 方法在用空集合调用时不是抛出异常，只是返回 null。因此新的方法更适合容易出现异常条件的情况。

**peek，element区别：**

element() 和 peek() 用于在队列的头部查询元素。与 remove() 方法类似，在队列为空时， element() 抛出一个异常，而 peek() 返回 null。

## Set

![img](pic/pic2)

### Set常用方法

```java
public interface Set<E> extends Collection<E> {
	int size();
    int hashCode();
    void clear();
    boolean equals(Object o);
    boolean isEmpty();
    boolean add(E e);
    boolean addAll(Collection<? extends E> c);
    boolean remove(Object o);
    boolean removeAll(Collection<?> c);
    boolean contains(Object o);
    boolean containsAll(Collection<?> c);
    boolean retainAll(Collection<?> c); 
    Iterator<E> iterator();
    Object[] toArray();
    T[] toArray(T[] a);
}
```

### HashSet基本操作

```java
public class HashSetTest {
    public static void main(String[] agrs){
        //创建HashSet集合：
        Set<String> hashSet = new HashSet<String>();
        System.out.println("HashSet初始容量大小："+hashSet.size());
        
        //元素添加：
        hashSet.add("...");
        System.out.println("HashSet容量大小："+hashSet.size());
        
        //迭代器遍历：
        Iterator<String> iterator = hashSet.iterator();
        while (iterator.hasNext()){
            String str = iterator.next();
            System.out.println(str);
        }
        
        //增强for循环
        for(String str:hashSet){
            ...;
        }
        
        //元素删除：
        hashSet.remove("...");
        
        //集合判断：
        boolean isEmpty = hashSet.isEmpty();
        boolean isContains = hashSet.contains("...");
    }
}
```

### TreeSet基本操作

```java
public class TreeSetTest {
    public static void main(String[] agrs){
        TreeSet<String> treeSet = new TreeSet<String>();
        System.out.println("TreeSet初始化容量大小："+treeSet.size());

        //元素添加：
        treeSet.add("...");

        //增加for循环遍历：
        for(String str:treeSet){
            ...;
        }

        //迭代器遍历：升序
        Iterator<String> iteratorAesc = treeSet.iterator();
        while(iteratorAesc.hasNext()){
            String str = iteratorAesc.next();
            System.out.println("遍历元素升序："+str);
        }

        //迭代器遍历：降序
        Iterator<String> iteratorDesc = treeSet.descendingIterator();
        while(iteratorDesc.hasNext()){
            String str = iteratorDesc.next();
            System.out.println("遍历元素降序："+str);
        }

        //元素获取:实现NavigableSet接口
        String firstEle = treeSet.first();//获取TreeSet头节点：

        // 获取指定元素之前的所有元素集合：(不包含指定元素)
        SortedSet<String> headSet = treeSet.headSet("...");

        //获取给定元素之间的集合：（包含头，不包含尾）
        SortedSet subSet = treeSet.subSet("...","...");

        //集合判断：
        boolean isEmpty = treeSet.isEmpty();
        boolean isContain = treeSet.contains("...");

        //元素删除：
        boolean removed = treeSet.remove("...");

       //删除并返回第一个元素：如果set集合不存在元素，则返回null
        String pollFirst = treeSet.pollFirst();

        //删除并返回最后一个元素：如果set集合不存在元素，则返回null
        String pollLast = treeSet.pollLast();
        
        //清空集合:
        treeSet.clear();
    }
}
```

## Map基本操作

```java
public interface Map<K, V>{
	void clear();
    int size();
    int hashCode();
	V put(K key, V value);
	V putAll(Map<? extends K, ? extends V> m);
    V get(Object key);
	V remove(Object key);
	boolean containsKey(Object key);
	boolean containsValue(Object value);
	boolean equals(Object o);
	boolean isEmpty();
	Set<Map.Entry<K, V>> entrySet();
	/*return a set view of the mappings contained in this map*/
	Set<K> keySet();
	/*return a set view of the keys contained in this map*/
	Collection<V> values();
	/*return a Collection view of the values contained in this map*/
}
```
```java
public interface Map.Entry<K, V>{
    boolean equals(Object o);
    K getKey();
    V getValue();
    int hashCode();
    V setValue(V value);
}
```

