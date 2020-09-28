# Hadoop组件

[TOC]

## 编程读取HDFS

得到与HDFS接口的`FileSystem`对象：

```java
Configuration conf=new Configuration();
FileSystem hdfs=FileSystem.get(conf);
```

要得到一个专用于本地文件系统的`FileSystem`对象，可用`factory`方法：

```java
FileSystem local=FileSystem.getLocal(conf);
```

得到一个目录中的文件列表：

```java
Path inputDir=new Path(args[0]);
FileStatus[] inputFiles=local.listStatus(inputDir);
```

数组`inputFiles`的长度等于指定目录中的文件个数。在`inputFiles`每一个`FileStatus`对象有元数据信息，如文件长度、权限、修改时间等。可以通过`FSDataInputStream`对象访问这个`Path`来读取文件。

```java
FSDataInputStream in = local.open(inputFiles[i].getPath());
byte buffer[] = new byte[256];
int bytesRead = 0;
while((bytesRead=in.read(buffer))>0){
    ...
}
in.close();
```

类似地有一个`FSDataOutputStream`对象用于将数据写入HDFS文件：

```java
Path hdfsFile=new Path(args[1]);
FSDataOutputStream out=hdfs.create(hdfsFile);
out.write(buffer, 0, bytesRead);
out.close();
```

下面的程序会逐一读取`inputFiles`中的所有文件，并写入目标HDFS文件：

```java
Configuration conf=new Configuration();
FileSystem hdfs=FileSystem.get(conf);
FileSystem local=FileSystem.getLocal(conf);
//输入输出目录
Path inputDir=new Path(args[0]);
Path hdfsFile=new Path(args[1]);
try{
    //得到本地文件列表
    FileStatus[] inputFiles=local.listStatus(inputDir);
    //生成HDFS输出流
    FSDataOutputStream out=hdfs.create(hdfsFile);
    for(int i=0; i<inputFiles.length; i++){
        System.out.println(inputFiles[i].getPath().getName());
        //打开本地输入流
        FSDataInputStream in=local.open(inputFiles[i].getPath());
        byte[] buffer=new byte[256];
        int bytesRead=0;
        while((bytesRead=in.read(buffer))>0){
            out.write(buffer, 0, bytesRead);
        }
        in.close();
    }
    out.close();
}catch(IOException e){
    e.printStackTrace();
}
```

## Hadoop的数据类型

基本类型不能用于MapReduce框架。为了让键值对可以在集群上移动，MR框架提供了一种序列化的方法，只有支持这种序列化的类才能用于该框架。

若需要自定义数据类型，那么需要实现`WritableComparable`接口，Hadoop内置一些实现了该接口的预定义类，如下：

| 类                | 描述                  |
| ----------------- | --------------------- |
| `BooleanWritable` | 标准布尔变量的封装    |
| `ByteWritable`    | 单字节数的封装        |
| `DoubleWritable`  | 双字节的封装          |
| `FloatWritable`   | 浮点数的封装          |
| `IntWritable`     | 整数的封装            |
| `LongWritable`    | Long的封装            |
| `Text`            | 使用UT8格式的文本封装 |
| `NullWritable`    | 无键值时的占位符      |

下面自定义一个数据类型，用来表示一个网络的边界，也可能表示两个城市之间额航线：

```java
public class Edge implements WritableComparable<Edge>{
    private String departureNode;
    private String arrivalNode;
    public String getDepartureNode(){return departureNode;}
    
    //说明如何读入数据
    @Override
    public void readFields(DataInput in) throws IOException{
        departureNode=in.readUTF();
        arrivalNode=in.readUTF();
    }
    //说明如何写出数据
    @Override
    public void write(DataOutput out) throws IOException{
        out.writeUTF(departureNode);
        out.writeUTF(arrivalNode);
    }
    //定义数据排序
    @Override
    public int compareTo(Edge o){
        return (departureNode.compareTo(o.departureNode)!=0)?departureNode.compareTo(o.departureNode):arrivalNode.compareTo(o.arrivalNode);
    }
}
```

## `Partitioner`：重定向Mapper输出

使用多个reducer时，我们需要采取一些方法确定mapper应该把键值对输出给谁。默认的做法是对键进行散列来确定reducer。

假设使用前面定义的`Edge`类来分析航班信息来决定从各个机场离港的乘客数目，这些数据为：

```shel
(San Francisco, Los Angeles)	Chuck Lam
(San Francisco, Dallas)			James Warren
```

如果使用默认的策略，那么这两行将被送到不同的reducer，但是我们是从机场离港这个角度计算的，那么这两行应该被送到同一个reducer。

下面是定制对应的`partitioner`：

```java
public class EdgePartitioner implements Partitioner<Edge, Writable>{
    @Override
    public int getPartition(Edge key, Writable value, int numPartitions){
        return key.getDepartureNode().hashCode()%numPartitions;
    }
    @Override
    public void configure(JobConf conf) {}
}
```

## 读和写

### InputFormat

Hadoop分割与读取数据输入文件的方式被定义在`InputFormat`接口的一个实现中，`TextInputFormat`是`InputFormat`的默认实现。`TextInputFormat`返回的键为每行的字节偏移量。

#### 常用的INPUTFORMAT类：

| InputFormat                    | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| `TextInputFormat`              | 在文本文件中每一个行为一个记录，key为一行的字节偏移，value为一行的内容<br>`key:LongWritable; value:Text` |
| `KeyValueTextInputFormat`      | 在文本文件中的每一行均为一个记录，以每行的第一个分隔符为界，分隔符之前为key，之后为value。分离器在属性`key.value.separator.in.input.line`中设定，默认为制表符`\t`<br>`key:Text; value:Text` |
| `SequenceFileInputFormat<K,V>` | 用于读取序列文件的`InputFormat`。键和值用户自定义，序列文件为Hadoop专用的压缩二进制文件格式（序列文件）。它专用于一个`MapReduce`作业和其他`Mapreduce`作业之间传递数据。<br>`key:K; value:V` |
| `NLineInputFormat`             | 与`TextInputFormat`相同，但每个分片一定一定有N行。N在属性`mapred.line.input.format.linespermap`中设定。默认为1。<br>`key:LongWritable; value:Text` |

#### 生成一个定制的InputFormat——InputSplit和RecordReader

有时会期望采用与标准`InputFormat`类不同的方式读取输入数据。这时你必须编写自定义的`InputFormat`类。`InputFormat`接口定义如下：

```java
public interface InputFormat<K, V>{
    //确定所有用于输入数据的文件，并将之分割为输入分片。每个map任务分配一个分片
    InputSplit[] getSplits(JobConf job, int numSplits) throws IOException;
    //提供RecordReader，循环提取给定分片中的记录，并解析每个记录为预定义类型的键和值
    RecordReader<K, V> getRecordReader(InputSplit split, JobConf job, Reporter reporter) throws IOException;
}
```

在创建子集的`InputFormat`类时，最好从负责文件分割的`FileInputFormat`类中继承一个子类。`FileInputFormat`实现了`getSplit`方法，保留了`getRecordReader`抽象让子类填写。`FileInputFormat`中实现的`getSplit`把输入数据粗略地划分为一组分片，分片数目在`numSplits`中限定，且每个分片大小必须大于`mapred.min.split.size`个字节，但小于文件系统的块。**在HDFS中默认为64MB。**

`RecordReader`负责把一个输入分片解析成记录，再把每个记录解析为一个键值对:

```java
public interface RecordReader<K, V>{
    boolean next(K key, V value) throws IOException;
    K createKey();
    V createValue();
    
    long getPos() throws IOException;
    public void close() throws IOException;
    float getProgress() throws IOException;
}
```

Hadoop内置了一些`RecordReader`实现类。比如，`LineRecordReader`实现`RecordReader<LongWritable, Text>`，被用于`TextInputFormat`；`KeyValueLineRecordReader`被用于`KeyValueTextInputFormat`中。

下面是自定义`InputFormat`类的一个用例，它按照特定的类型读取记录，而不是使用普通的Text类型。这个类最终会把时间戳和URL都作为Text类

```java
public class TimeUrlTextInputFormat extends FileInputFormat<Text, URLWritable>{
    public RecordReader<Text, URLWritable> getRecordReader(InputSplit input, JobConf job, Reporter reporter) throws IOException{
        return new TimeUrlLineRecordReader(job, (FileSplit) input);
    }
    public class URLWritable implements Writable{
        protected URL url;
        public URLWritable() {}
        public URLWritable(URL url){
            this.url=url;
        }
        public void write(DataOutput out) throws IOException{
            out.writeUTF(url.toString());
        }
        public void readFields(DataInput in) throws IOException{
            url=new URL(in.readUTF());
        }
        public void set(String s) throws MalformedURLException{
            url=new URL(s);
        }
    }
}
```

```java
class TimeUrlLineRecordReader implements RecordReader<Text, URLWritable>{
    private KeyValueLineRecordReader lineReader;
    //疑惑，书上没解释这两变量的用处
    private Text lineKey, lineValue;
    public TimeUrlLineRecordReader(JobConf job, FileSplit split) throws IOException{
        lineReader=new KeyValueLineRecorderReader(job, split);
        lineKey=lineReader.createKey();
        lineValue=lineReader.createValue();
    }
    //next方法主要把URLWritable对象转换成Text对象
    public boolean next(Text key, URLWritable value) throws IOException{
        if(!lineReader.next(lineKey, lineValue)){
            return false;
        }
        key.set(lineKey);
        value.set(lineValue.toString());
        return true;
    }
    public Text createKey(){
        return new Text("");
    }
    public URLWritable createValue(){
        return new URLWritable();
    }
    public long getPos() throws IOException{
        return lineReader.getPos();
    }
    public float getProgress() throws IOException{
        return lineReader.getProgress();
    }
    public void close() throws IOException{
        lineReader.close();
    }
}
```

### OutputFormat

当MR输出数据到文件时，使用的是`OutputFormat`类。输出文件放在一个公用目录中，通常命名为`part-nnnnn`，这里的`nnnnn`是reducer的分区ID。`RecordWriter`对象将输出结果进行格式化，而`RecordReader`对输入格式进行解析。

下面是Hadoop内置的实现：

| `OutputFormat`                   | 描述                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| `TextOutputFormat<K, V>`         | 将每个记录写为一行文本。键和值以字符串的形式写入，并以制表符（`\t`）分隔，这个分隔符可以在属性`mapred.textoutputformat.separator`中修改<br /> |
| `SequenceFileOutputFormat<K, V>` | 以Hadoop专有序列文件格式写入键值对。与`SequenceFileInputFormat`配合使用 |
| `NullOutputFormat<K, V>`         | 无输出                                                       |

> 注意，默认的`OutputFormat`是`TextOutputFormat`

`TextOutputFormat`采用可被`KeyValueTextInputFormat`识别的格式输出数据。如果把键的类型设为`NullWritable`，也可以采用可被`TextInputFormat`识别的输出格式。`SequenceFileOutputFormat`以序列文件格式输出数据，使其可以通过`SequenceInputFormat`来读取，有助于通过中间数据结果将MR作业串接起来。


