# Hadoop配置

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
    <value>/opt/hadoop-3.2.1/tmp</value>
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

mapred-env.xml:

```xml
<configuration>
  <!-- 指定MR运行在Yarn上 -->
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
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

如果集群是第一次启动，需要格式化NameNode，注意这应该是最后一步，在所有配置文件配置完之后格式化，并且在格式化之前先要把停止所有namenode和datanode进程，以及删除/opt/hadoop-3.2.1/data和/opt/hadoop-3.2.1/logs/*。

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

