# MapReduce例子

## 自定义序列化

统计手机用户流量日志，日志内容为：`phoneNumber+'\t'+upFlow+'\t'+downFlow`。

输出内容为`phoneNumber+'\t'+upFlow+'\t'+downFlow+'\t'+sumFlow`

```java
public static class FlowWritable implements Writable {
        private int upFlow;
        private int downFlow;
        private int sumFlow;

        public FlowWritable() {
        }

        public FlowWritable(int upFlow, int downFlow) {
            this.upFlow = upFlow;
            this.downFlow = downFlow;
            this.sumFlow = upFlow + downFlow;
        }
    	//此处省略get/set方法

        @Override
        public void write(DataOutput dataOutput) throws IOException {
            dataOutput.writeInt(upFlow);
            dataOutput.writeInt(downFlow);
            dataOutput.writeInt(sumFlow);
        }

        @Override
        public void readFields(DataInput dataInput) throws IOException {
            upFlow = dataInput.readInt();
            downFlow = dataInput.readInt();
            sumFlow = dataInput.readInt();
        }

        @Override
        public String toString() {
            return upFlow + "\t" + downFlow + "\t" + sumFlow;
        }
    }
```

```java
    public static class FlowWritableMapper extends Mapper<Object, Text, Text, FlowWritable> {
        @Override
        protected void map(Object key, Text value, Context context) throws IOException, InterruptedException {
            String[] split = value.toString().split("\t");
            Text phone = new Text(split[0]);
            FlowWritable flow = new FlowWritable(Integer.parseInt(split[1]), Integer.parseInt(split[2]));
            context.write(phone, flow);
        }
    }
```

```java
    public static class FlowWritableReducer extends Reducer<Text, FlowWritable, Text, FlowWritable> {
        @Override
        protected void reduce(Text key, Iterable<FlowWritable> values, Context context) throws IOException, InterruptedException {
            int upFlow = 0, downFlow = 0;
            for (FlowWritable value : values) {
                upFlow += value.getUpFlow();
                downFlow += value.getDownFlow();
            }
            context.write(key, new FlowWritable(upFlow, downFlow));
        }
    }
```

```java
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "flow statistics");
        job.setJarByClass(FlowStatistics.class);
        job.setMapperClass(FlowWritableMapper.class);
        job.setReducerClass(FlowWritableReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowWritable.class);
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 1 : 0);
    }
```

## 数据去重

输入文件格式:`date+'\t'+count`。

现只需要`date`数据，需要去重，我们只需要把需要去重的属性设为key，value设为`NullWritable`。

```java
public static class DateDistinctMapper extends Mapper<Object, Text, Text, NullWritable> {		
		public void map(Object key, Text value, Context context ) 
				throws IOException, InterruptedException {
	    	String[] strs = value.toString().split(" ");
	    	Text date = new Text(strs[0]);//取到日期作为key
			context.write(date, NullWritable.get());
	    }
	}
  
public static class DateDistinctReducer extends Reducer<Text,NullWritable,Text,NullWritable>{
    
		public void reduce(Text key, Iterable<NullWritable> values, Context context) 
				throws IOException, InterruptedException {
			context.write(key, NullWritable.get());
	    }
	}
```

## 数据排序

+ 单列排序
  + 可以利用Shuffle默认对key排序的规则
  + 自定义继承`WritableComparator`的排序类，实现`compare`方法
+ 二次排序
  + 实现可序列化的比较类`WritableComparator<T>`，并实现`compareTo`方法

### 日期降序

实现自定义的排序比较器：

```java
public static class MyComparator extends WritableComparator {
		public MyComparator() {
			// TODO Auto-generated constructor stub
			super(IntWritable.class, true);
		}

		@Override
		@SuppressWarnings({ "rawtypes", "unchecked" }) // 不检查类型
		public int compare(WritableComparable a, WritableComparable b) {
			// CompareTo方法，返回值为1则降序，-1则升序
			// 默认是a.compareTo(b)，a比b小返回-1，现在反过来返回1，就变成了降序
			return b.compareTo(a);
	}
```

在`main`函数中指定自定义的排序比较器：

```java
job.setSortComparatorClass(MyComparator.class);
```

### 手机流量按上行流量升序，下行流量降序

```java
public static class MySortKey implements WritableComparable<MySortKey> {
		private int upFlow;
		private int downFlow;
		private int sumFlow;

		public FlowSort(int up, int down) {
			upFlow = up;
			downFlow = down;
			sumFlow = up + down;
		}

		@Override
		public void write(DataOutput out) throws IOException {
			// TODO Auto-generated method stub
			out.writeInt(upFlow);
			out.writeInt(downFlow);
			out.writeInt(sumFlow);
		}

		@Override
		public void readFields(DataInput in) throws IOException {
			// TODO Auto-generated method stub
			upFlow = in.readInt();
			downFlow = in.readInt();
			sumFlow = in.readInt();
		}

		@Override
		public int compareTo(MySortKey o) {
			if ((this.upFlow - o.upFlow) == 0) {// 上行流量相等，比较下行流量
				return o.downFlow - this.downFlow;// 按downFlow降序排序
			} else {
				return this.upFlow - o.upFlow;// 按upFlow升序排
			}
		}

		@Override
		public String toString() {
			// TODO Auto-generated method stub
			return upFlow + "\t" + downFlow + "\t" + sumFlow;
		}
	}

	public static class SortMapper extends Mapper<Object, Text, MySortKey, Text> {
		Text phone = new Text();
		MySortKey mySortKey = new MySortKey();

		public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
			String[] lists = value.toString().split("\t");
			phone.set(lists[0]);
			mySortKey.setUpFlow(Integer.parseInt(lists[1]));
			mySortKey.setDownFlow(Integer.parseInt(lists[2]));
			context.write(mySortKey, phone);// 调换手机号和流量计数，后者作为排序键
		}
	}

	public static class SortReducer extends Reducer<MySortKey, Text, Text, MySortKey> {
		public void reduce(MySortKey key, Iterable<Text> values, Context context)
				throws IOException, InterruptedException {
			for (Text value : values) {
				System.out.println(value.toString()+","+key.toString());
				context.write(value, key);// 再次把手机号和流量计数调换
			}
		}
	}
```

## 自定义分区

仍以上述的手机用户流量日志为例，在上个例子中的统计需要基础上添加一个新需求：不同省份的手机号放到不同的文件里。我们可以自定义分区。

> 注意，一个分区对应一个reduce任务，一个reduce任务对应一个输出文件

```java
// 自定义分区类
public static class PhoneNumberPartitioner extends Partitioner<Text, FlowWritable> {
		private static HashMap<String, Integer> numberDict = new HashMap<>();
		static {
			numberDict.put("133", 0);
			numberDict.put("135", 1);
			numberDict.put("137", 2);
			numberDict.put("138", 3);
		}

		@Override
		public int getPartition(Text key, FlowWritable value, int numPartitions) {
			String num = key.toString().substring(0, 3);
			// 借助HashMap返回不同手机段对应的分区号
             return numberDict.getOrDefault(num, 4);
			// 也可以直接通过if判断，如
			// 根据手机段对数据进行分区，返回不同分区号
			// if (key.toString().startsWith("133")) return 0 % numPartitions;
             // if (key.toString().startsWith("135")) return 1 % numPartitions;
		}
	}
```

```java
// 设置分区类,及Reducer数目
job.setPartitionerClass(PhoneNumberPartitioner.class);
job.setNumReduceTasks(4);
```

## 分组

比如有订单数据，其格式为：`订单id\t商品id\t成交金额`。

需要求出每个订单中成交金额最大的一笔交易。

通常做法是将订单id作为key，后两者作为value，在reduce中取出value遍历取最值即可。

这也可以使用分组的技术。

先定义一个序列化对象Pair，用来存`order_id, amount`，Mapper端输出类似：

| key                 | value |
| ------------------- | ----- |
| {order_id,  amount} | null  |

通过Pair中的order_id分组，因为Pair又是可比较的，设置同一组按照amount降序排列。后再Reduce端取第一个键值对即可。Reduce端输入格式如下：

| key                                | value |
| ---------------------------------- | ----- |
| {order_id, [amount1, amount2, ...} | null  |

```java
public static class Pair implements WritableComparable<Pair> {
		private String order_id;
		private DoubleWritable amount;

		public Pair() {
			// TODO Auto-generated constructor stub
		}

		public Pair(String id, DoubleWritable amount) {
			this.order_id = id;
			this.amount = amount;
		}

		// 省略get/set

		@Override
		public void write(DataOutput out) throws IOException {
			// TODO Auto-generated method stub
			out.writeUTF(order_id);
			out.writeDouble(amount.get());
		}

		@Override
		public void readFields(DataInput in) throws IOException {
			// TODO Auto-generated method stub
			order_id = in.readUTF();
			amount = new DoubleWritable(in.readDouble());
		}

		@Override
		public int compareTo(Pair o) {
			if (order_id.equals(o.order_id)) {// 同一order_id，按照amount降序排序
				return o.amount.compareTo(amount);
			} else {
				return order_id.compareTo(o.order_id);
			}
		}

	}
```

```java
public static class GroupComparator extends WritableComparator {
		public GroupComparator() {
			// 注册比较方法         
			super(Pair.class, true);
		}
		// Mapper端会对Pair排序，之后分组的规则是对Pair中的order_id比较
		@Override
		public int compare(WritableComparable a, WritableComparable b) {
			// TODO Auto-generated method stub
			Pair oa = (Pair) a;
			Pair ob = (Pair) b;
			return oa.getOrder_id().compareTo(ob.getOrder_id());
		}
	}
```

```java
public static class MyMapper extends Mapper<Object, Text, Pair, NullWritable> {
		Pair pair = new Pair();

		public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
			String[] strs = value.toString().split(" ");
			pair.setOrder_id(strs[0]);
			pair.setAmount(new DoubleWritable(Double.parseDouble(strs[2])));
			context.write(pair, NullWritable.get());// 道理同上，以Pair作为key
			System.out.println(pair.getOrder_id()+","+pair.getAmount());
		}
	}
```

```java
public static class MyReducer extends Reducer<Pair, NullWritable, Text, DoubleWritable> {
		public void reduce(Pair key, Iterable<NullWritable> values, Context context)
				throws IOException, InterruptedException {
			context.write(new Text(key.getOrder_id()), key.getAmount());// 已经排好序的，取第一个即可
			System.out.println(key.order_id+": "+key.amount.get());
		}
	}
```

## 多文件输入输出及不同输入输出格式化类型

### 合并多个小文件

要计算的目录文件中有大量的小文件，会造成分配任务和资源的开销比实际的计算开销还大，这就产生了效率损耗。需要先把一些小文件合并成一个大文件。

我们是合并小文件，没必要切片，直接将文件对象视为一个分片，键值对以文件名为key，文件对象为value。

```java
//将整个文件作为一条记录处理
public class WholeFileInputFormat extends FileInputFormat<NullWritable,Text>{
 
	//表示文件不可分
	@Override
	protected boolean isSplitable(JobContext context, Path filename) {
		return false;
	}
 
	@Override
	public RecordReader<NullWritable, Text> createRecordReader(
			InputSplit split, TaskAttemptContext context) throws IOException,
			InterruptedException {
		WholeRecordReader reader=new WholeRecordReader();
		reader.initialize(split, context);
		return reader;
	}
  
}
```

```java
//实现RecordReader,为自定义的InputFormat服务
public class WholeRecordReader extends RecordReader<NullWritable,Text>{
 
	private FileSplit fileSplit;
	private Configuration conf;
	private Text value=new Text();
	private boolean processed=false;//表示记录是否被处理过
	@Override
	public NullWritable getCurrentKey() throws IOException,
			InterruptedException {
		return NullWritable.get();
	}
 
	@Override
	public Text getCurrentValue() throws IOException,
			InterruptedException {
		return value;
	}
 
	@Override
	public float getProgress() throws IOException, InterruptedException {
		 return processed? 1.0f : 0.0f;
	}
	@Override
	public void initialize(InputSplit split, TaskAttemptContext context)
			throws IOException, InterruptedException {
		this.fileSplit=(FileSplit)split;
	    this.conf=context.getConfiguration();
	}
 
	@Override
	public boolean nextKeyValue() throws IOException, InterruptedException {
		if(!processed)
		{
			byte[]contents=new byte[(int)fileSplit.getLength()];
			Path file=fileSplit.getPath();
			FileSystem fs=file.getFileSystem(conf);
			FSDataInputStream in=null;
			try{
				in=fs.open(file);
				IOUtils.readFully(in, contents, 0, contents.length);
			    value.set(contents,0,contents.length);
			}
			finally{
				IOUtils.closeStream(in);
			}
			processed=true;
			return true;
		}
		return false;
	}
 
	@Override
	public void close() throws IOException {
		// TODO Auto-generated method stub
		
	}
 
}
```

```java
public class SmallFilesToSequenceFileConverter{
	private static class SequenceFileMapper extends Mapper<NullWritable,Text,Text,Text>
	{
		private Text filenameKey;
		//setup在task之前调用，用来初始化filenamekey
		
		@Override
		protected void setup(Context context)
				throws IOException, InterruptedException {
			InputSplit split=context.getInputSplit();
		    Path path=((FileSplit)split).getPath();
		    filenameKey=new Text(path.toString());
		}
		
		@Override
		protected void map(NullWritable key,Text value,Context context)
				throws IOException, InterruptedException {
			context.write(filenameKey, value);
		}
	}
	
	public static void main(String[] args) throws Exception {
		Configuration conf=new Configuration();
		Job job=Job.getInstance(conf,"SmallFilesToSequenceFileConverter");
		
		job.setJarByClass(SmallFilesToSequenceFileConverter.class);
		
		job.setInputFormatClass(WholeFileInputFormat.class);
		//job.setOutputFormatClass(TextOutputFormat.class);
		
		
		job.setMapperClass(SequenceFileMapper.class);
		
		//设置最终的输出
	    job.setOutputKeyClass(Text.class);
	    job.setOutputValueClass(Text.class);
	    
		FileInputFormat.addInputPath(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job,new Path(args[1]));
	
	    System.exit(job.waitForCompletion(true) ? 0:1);
	}
}

```

