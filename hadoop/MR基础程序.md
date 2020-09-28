# MR基础程序

## 专利数据集

本文所使用的数据集是专利引用数据集`cite75_99.txt`和专利描述数据集`apat63_99.txt`，可在http://www.nber.org/patents上取得。

### 专利引用数据

格式为：`"CITING（引用）", "CITED（被引）"`

数据集采用标准的逗号分隔取值（CSV）格式。

### 专利描述数据

格式为：

`"PATENT","GYEAR","GDATE","APPYEAR","COUNTRY","POSTATE","ASSIGNEE",`

`"ASSCODE","CLAIMS","NCLASS","CAT","SUBCAT","CMADE","CRECEIVE",`

`"RATIOCIT","GENERAL","ORIGINAL","FWDAPLAG","BCKGTLAG","SELFCTUB",`

`"SELFCTLB","SECDUPBD","SECDLWBD"`

前10个属性描述如下：

| 属性名     | 内容                         |
| ---------- | ---------------------------- |
| `PATENT`   | 专利号                       |
| `GYEAR`    | 批准年                       |
| `GDATE`    | 批准日                       |
| `APPYEAR`  | 申请年                       |
| `COUNTRY`  | 国家                         |
| `POSTATE`  | 所在州                       |
| `ASSIGNEE` | 专利拥有者数字标识           |
| `ASSCODE`  | 1位（1~9）表示的专利权人类型 |
| `CLAIMS`   | 声明数目                     |
| `NCLASS`   | 3位表示的主要专利类型        |

## 构建MR程序的基础模板

第一个程序将读取专利引用数据并对它进行侧排。对每一个专利，我们希望找到那些引用它的专利并进行合并，输出如下：

`1000067 5312208,4944640,50717294`

表示专利5312208,4944640,50717294引用了1000067。

```java
public class MyJob extends Configured implements Tool{
    public static class MapClass extends MapReduceBase implements Mapper<Text, Text, Text, Text>{
        public void map(Text key, Text value, OutputCollector<Text, Text> output, Reporter reporter) throws IOException{
            //注意这反向了
            output.collect(value, key);
        }
    }
    public static class Reduce extends MapReduceBase implements Reducer<Text, Text, Text, Text>{
        public void reduce(Text key, Iterator<Text> values, OutputCollector<Text, Text> output, Reporter reporter) throws IOException{
            String cav="";
            while(values.hasNext()){
                if(csv.length()>0) csv+=",";
                csv+=values.next().toString();
            }
            output.collect(key, new Text(csv));
        }
    }
    public int run(String[] args){
        Configuration conf=getConf();
        JobConf job=new JobConf(conf, MyJob.class);
        Path in=new Path(args[0]);
        Path out=new Path(args[1]);
        FileInputFormat.setInputPaths(job, in);
        FileOutPutFormat.setOutputPath(job, out);
        job.setJobName("MyJob");
        job.setMapperClass(MapClass.class);
        job.setReducerClass(Reduce.class);
        job.setInputFormat(KeyValueTextInputFormat.class);
        job.setOutputFormat(TextOutputFormat.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);
        job.set("Key.value.separator.in.input.line",",");
        JobClient.runJob(job);
        return 0;
    }
    public static void main(String[] args) throws Exception{
        int res=ToolRunner.run(new Configuration(), new MyJob(), args);
        System.exit(res);
    }
}
```

`JobConf`对象有许多参数，默认是用Hadoop安装是的配置文件。同时，用户可能希望在命令行启动一个作业时传递额外的参数来改变作业配置。`Driver`可以通过自定义一组命令并自行处理用户参数，来支持用户修改其中的一些配置。Hadoop框架便提供了`ToolRunner、Tool、Configuration`来简化其实现。例如，我们用如下命令执行`MyJob`类：

```shell
hadoop jar MyJob.jar MyJob input output
```

如果我们运行作业仅仅是想看到mapper的输出，可以用选项`-D mapred.reduce.tasks=0 `将reducer的数目设置为0.

```shell
hadoop jar MyJob.jar MyJob -D mapred.reduce.tasks=0 input output
```

即使程序不能清楚理解`-D`选项也没关系，通过`ToolRunner， MyJob`可以自动支持下表的选项：

| 选项                           | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| `-conf <configuration file>`   | 指定一个配置文件                                             |
| `-D <property=value>`          | 给`JobConf`属性赋值                                          |
| `-fs <local|namenode:port>`    | 指定一个`NameNode`，可以是“local”                            |
| `-jt <local|jobtracker:port>`  | 指定一个`JobTracker`                                         |
| `-files <list of files>`       | 指定一个以逗号分隔的文件列表，用于MR作业。这些文件自动地分布到所有结点，使之可从本地获取 |
| `-libjars <list of jars>`      | 指定一个以逗号分隔的jar文件，使之包含在所有任务JVM的classpath中 |
| `-archives <list of archives>` | 指定一个以逗号分隔的存档文件列表，使之可以在所有任务结点上打开 |

Mapper类和Reducer类如下所示：

```java
public static class MapClass extends MapReduceBase implements Mapper<K1, V1, K2, V2>{
    public void map(K1 key, V1 value, OutputCollector<K2, V2> output, Reporter reporter) throws IOException { }
}
public static class Reduce extends MapReduceBase implements Reducer<K2, V2, K3, V3>{
    public void reduce(K2 key, Iterator<V2> values, OutputCollector<K3, V3> output, Reporter reporter) throws IOException {}
}
```

每个`map()`方法的调用分别被授予一个类型为K1和V1的键值对，有mapper生成，并通过`OutputCollector`对象的`collect()`方法来输出，需要在合适位置调用：

```java
output.collect((K2) k, (V2) v);
```

在Reducer中`reduce()`方法的每次调用均被赋予K2类型的键，以及V2类型的一组值。注意它必须与Mapper中使用的K2和V2类型相同。`Reduce()`可以循环遍历V2类型的所有值：

```java
while(values.hasNext()){
    V2 v=values.next();
}
```

`Reduce()`方法还使用`OutputCollector`来搜集其键值的输出，类型为K3/V3。在`reduce()`方法中可以调用：

```java
output.collect((K3) k, (V3) v);
```

所有的键与值的类型必须是`Writable`的子类型，来确保Hadoop的序列化接口可以把数据在分布式集群上发送。事实上，键的类型实现了`WritableComparable`，它是`Writable`的接口。键的类型还需额外支持`compareTo()`方法，因为在MR框架中键会被用来进行排序。

## 计数

我们希望从专利引用数据集中得到专利被引用的次数，格式为：`id \t count `。

只需要在上述程序上修改`Reducer`：

```java
public static class Reduce extends MapReduceBase implements Reducer<Text, Text, Text, IntWritable>{
    public void reduce(Text key, Iterator<Text> values, OutputCollector<Text, IntWritable> output, Reporter reporter) throws IOException{
        int count=0;
        while(values.hasNext()){
            values.next();
            count++;
        }
        output.collect(key, new IntWritable(count));
    }
}
```

计算不同引用次数专利的数目：

```java
public class CitationHistogram extends Configured implements Tool{
    public static class MapClass extends MapReduceBase implements Mapper<Text, Text, IntWritable, IntWritable>{
        private final static IntWritable uno=new IntWritable(1);
        private IntWritable citationCount=new IntWritable();
        public void map(Text key, Text value, OutputCollector<IntWritable, IntWritable> output, Reporter reporter) throws IOException{
            citationCount.set(Integer.parseInt(value.toString()));
            output.collect(citationCount, uno);
        }
    }
    public static class Reduce extends MapReduceBase implements Reducer<IntWritable, IntWritable, IntWritable, IntWritable>{
        public void reduce(IntWritable key, Iterator<IntWritable> values, OutputCollector<IntWritable, IntWritable> output, Reporter reporter) throws IOException{
            int count=0;
            while(values.hasNext()){
                count+=values.next().get();
            }
            output.collect(key, new IntWritable(count));
        }
    }
}
```

> 注意，有多少记录，`map()`方法就会被调用多少次（对每个JVM而言，就是一个分片中的记录数）。减少在`map()`中生成的对象个数可以提高性能，并减少垃圾回收。

## 使用combiner提升性能

