# Hadoop配置

[TOC]

### centos7网络配置

```shell
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

```shell
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=560d28c7-e216-49ec-9163-3694a2a20c9e
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.200.131
GATEWAY=192.168.200.2
NETMASK=255.255.255.0
DNS1=8.8.8.8
DNS2=61.147.37.1
```

```shell
vi /etc/resolv.conf
```

```shell
nameserver 8.8.8.8
nameserver 61.147.37.1
```

```shell
service network restart
```

```shell
#如果需要外界工具连接centos7，那么还需要关闭防火墙
#临时关闭防火墙
systemctl stop firewalld
#禁止防火墙开机启动
systemctl disable firewalld
```

### 分布式配置

#### 集群部署规划

|      | master               | slave1                         | slave2                        |
| ---- | -------------------- | ------------------------------ | ----------------------------- |
| HDFS | NameNode<br>DataNode | DataNode                       | SecondaryNameNode<br>DataNode |
| YARN | NodeManager          | ResourceManager<br>NodeManager | NodeManager                   |

#### SSH免密登录配置

```shell
#生成公钥与私钥
ssh-keygen -t rsa
#将公钥拷贝到要免密登录的目标机器上
ssh-copy-id master
ssh-copy-id slave1
ssh-copy-id slave2
```

#### 集群分发脚本

```bash
#!/bin/bash
#获取输入参数个数，如果没有参数，直接退出
pcount=$#
if((pcount==0)); then
echo no args;
exit;
fi
#获取文件名称
p1=$1
fname=`basename $p1`
echo fname=$fname
#获取上级目录到绝对路径
pdir=`cd -P $(dirname $p1); pwd`
echo pdir=$pdir

#同步
echo --------------------hadoop slave1---------------------
rsync -rvl $pdir/$fname root@slave1:$pdir
echo --------------------hadoop slave2---------------------
rsync -rvl $pdir/$fname root@slave2:$pdir
```

#### 核心配置文件

core-site.xml:

```xml
<configuration>
  <!-- 指定HDFS中的NameNode的地址 -->
  <property>
    <name>fs.default.name</name>
    <value>hdfs://master:9000</value>
  </property>
  <property>
    <!-- 指定Hadoop运行时产生文件的存储目录 -->
    <name>hadoop.tmp.dir</name>
    <value>/opt/hadoop-3.2.1/data/tmp</value>
  </property>
</configuration>
```

hadoop-env.sh:

```shell
export JAVA_HOME=/opt/jdk1.8.0_221
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```

hdfs-site.xml:

```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>3</value>
  </property>
  <!-- 指定Hadoop辅助名称节点主机配置 -->
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>slave2:50090</value>
  </property>
</configuration>
```

yarn-env.sh:

```shell
export JAVA_HOME=/opt/jdk1.8.0_221
```

yarn-site.xml:

```xml
<configuration>
  <!-- Site specific YARN configuration properties -->
  <!-- Reducer获取数据的方式 -->
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <!-- 指定YARN的ResourceManager的地址 -->
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>slave2</value>
  </property>
</configuration>
```

mapred-env.sh:

```shell
export JAVA_HOME=/opt/jdk1.8.0_221
```

mapred-site.xml:

```xml
<configuration>
  <!-- 指定MR运行在Yarn上 -->
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <!-- Hadoop3.2.1需要指定如下路径 -->
  <property>
    <name>yarn.app.mapreduce.am.env</name>
    <value>HADOOP_MAPRED_HOME=/opt/hadoop-3.2.1</value>
  </property>
  <property>
    <name>mapreduce.map.env</name>
    <value>HADOOP_MAPRED_HOME=/opt/hadoop-3.2.1</value>
  </property>
  <property>
    <name>mapreduce.reduce.env</name>
    <value>HADOOP_MAPRED_HOME=/opt/hadoop-3.2.1</value>
  </property>
</configuration>
```

配置集群信息，编辑/opt/hadoop-3.2.1/etc/hadoop/works:

```shell
master
slave1
slave2
```

之后在集群上分发配置好的文件。

如果集群是第一次启动，需要格式化NameNode，注意这应该是最后一步，在所有配置文件配置完之后格式化，并且在格式化之前先要把停止所有namenode和datanode进程，以及删除/opt/hadoop-3.2.1/data和/opt/hadoop-3.2.1/logs/*。（/data里记录着NameNode与DataNode的clusterId，若不一致则会出现问题）

```shell
bin/hdfs namenode -format
```

启动集群

```shell
sbin/start-dfs.sh
```

启动YARN

```shell
sbin/start-yarn.sh
```

> 注意，NameNode和ResourceManager如果不是同一台机器，那么不能再NameNode启动YARN，应在ResourceManager所在的机器启动YARN！

```shell
#查看集群状态
hadoop dfsadmin -report
```

#### 浏览器查看集群状态

`master:8042`:总览

`slave1:8080`:查看ResourceManager

`slave2:50090`：空白...这里配置的是SecondaryNameNode，推测当NameNode挂了之后才会显示信息？

#### 集群启动/停止方式总结

##### 各个服务组件逐一启动/停止

+ 分别启动/停止HDFS组件

  ```shell
  hadoop-daemon.sh start/stop namenode/datanode/secondarynamenode
  ```

+ 启动/停止YARN

  ```shell
  yarn-daemon.sh start/stop resourcemanager/nodemanager
  ```

##### 各个模块分开启动/停止（配置ssh是前提）

+ 整体启动/停止HDFS

  ```shell
  start-dfs.sh / stop-dfs.sh
  ```

+ 整体启动/停止YARN

  ```shell
  start-yarn.sh / stop-yarn.sh
  ```

#### 测试

```shell
hdfs dfs -mkdir /user/root/input
hdfs dfs -put ./test /user/root/input/
#自带的wordcount测试jar包
hadoop jar /opt/hadoop-3.2.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar wordcount /user/root/input /user/root/output
```

#### 配置历史服务器

配置`mapred-site.xml`：

```xml
  <!-- 历史服务器端地址 -->
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>master:10020</value>
  </property>
  <!-- 历史服务器web端地址 -->
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>master:19888</value>
  </property>
```

```shell
#启动历史服务器
sbin/mr-jobhistory-daemon.sh start historyserver
```

那么可以在`http://master:19888/jobhistory`上查看任务历史。

#### 配置日志的聚集

MapReduce是在各个机器上运行的，在运行过程中产生的日志存在于各个机器上，为了能够统一查看各个机器的运行日志，将日志集中存放在HDFS上，这个过程就是日志聚集。

配置`yarn-site.xml`:

```xml
  <!-- 日志聚集功能 -->
  <property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
  </property>
  <!-- 日志保留时间设置7天，单位秒 -->
  <property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>604800</value>
  </property>
```

#### 集群时间同步

##### 时间服务器配置

用master做时间服务器

```shell
yum install ntp
```

```shell
vi /etc/ntp.conf
```

修改内容如下：

```shell
#授权192.168.200.0-192.168.200.255网段上的所有机器可以从这台机器查询和同步时间
#restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap为
restrict 192.168.200.0 mask 255.255.255.0 nomodify notrap

#集群在局域网中，不使用其他互联网上的时间
#注释掉下面四行
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst

#当该节点丢失网络连接，依然可以采用本地时间作为时间服务器为集群中的其他节点提供时间同步
server 127.127.1.0
fudge 127.127.1.0 stratum 10
```

```shell
vi /etc/sysconfig/ntpd
```

增加以下内容：

```shell
#让硬件时间与系统时间一起同步
SYNC_HWCLOCK=yes
```

```shell
service ntpd start
#设置ntpd服务开机启动
chkconfig ntpd on
```

##### 其他机器配置

```shell
crontab -e
```

编写定时任务如下：

```shell
#每10分钟与时间服务器同步一次
*/10 * * * * /usr/sbin/ntpdate master
```

