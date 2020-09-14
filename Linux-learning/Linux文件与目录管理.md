# Linux文件与目录管理

### 目录相关操作

```shell
#显示当前目录
[root@yikang repos]# pwd
/var/lib/yum/repos
```

```shell
mkdir [-mp] dir
# -m:设置权限
# -p:递归创建目录
rmdir [-p] dir
# 注意只能删除空目录
# -p:连同上层的空目录一起删除
```

### 文件与目录管理

```shell
ls [options] [dir]
# -a:全部文件，包含隐藏文件(开头为.)
# -d:仅列出目录本身，不列出目录内的文件数据
# -h:易读方式
# -l:长数据串，包含文件的属性与权限等
# -S:以文件容量大小排序
# -t:以时间排序
```

```shell
cp [options] source destination
# -a:相当于-dR --preserve=all, 数据属性完全一致（修改时间，所有者等属性)
# -i:若目标文件已存在，在覆盖前会询问
# -p:连同文件的属性一起复制过去，而非使用默认属性（备份常用）
# -r:递归复制，常用于目录的复制
# -u:dest比source旧才更新dest
# -n:不覆盖已有文件
```

```shell
rm [options] file/dir
# -f: force。忽略不存在的文件，不会出现警告等信息
# -i: 互动模式
# -r: 递归删除
```

```shell
mv [options] source... destination
# -f: 若目标文件已存在，则直接覆盖
# -i: 互动模式
# -u: 若目标文件已存在，且source更新，则更新
```

```shell
#取文件名
[root@yikang ~]# basename /etc/sysconfig/network
network
#取目录名
[root@yikang ~]# dirname /etc/sysconfig/network
/etc/sysconfig
```

### 文件内容查阅

#### 直接查看文件内容

```shell
#全部打印
cat [options] filename
# -A: 可列出一些特殊字符，而不是空白(如换行与TAB）
# -b: 列出行号，空白行不标
# -n: 列出行号
# -E: 行末用$显示
```

```shell
#反向打印（指行）
tac [options] filename
```

#### 可翻页查看

```shell
more filename
# Enter:向下滚动一行
# Space:向下滚动一页
# b:向上翻页
# :f:显示文件名和当前行数
# q:退出
less filename
# 可用方向键滚动
# Space:向下翻一页
# PageDown:向下翻一页
# PageUp:向上翻一页
# /String:向下查询
# ?String:向上查询
# n:下一个查询
# N：上一个查询
# q：退出
```

#### 数据选取

```shell
head [-n number] filename
#显示前number行，number也可为负数
tail [-n number] filename
# -n:后number行
# -f:持续检测文件，若有数据写入，则立马输出至屏幕
```

#### 非文本文件

如果想查阅执行文件等非文本文件，需要用od命令。

```shell
od [-t TYPE] filename
# -t:后面可以接类型参数，如：
# a:用默认的字符输出
# c:使用ASCII字符输出
# d[size]:十进制输出，每个整数占用size bites
# f[size]:浮点数输出，每个数占用size bites
# o[size]:八进制输出，每个数占用size bites
# x[size]:十六进制，每个数占用size bites
```

#### 修改文件时间或创建新文件:touch

Linux下有三个主要的变动时间：

+ modification time（mtime）：当文件的内容变更时，该时间会更新
+ status time（ctime）：当文件的状态变更时，该时间会更新，例如权限与属性等变动
+ access time（atime）：当文件的内容被取用时，...。例如用cat读取文件

> ls命令默认显示出来的是mtime

```shell
ls -l filename
ls -l --time=atime filename
ls -l --time=ctime filename
```

```shell
touch [-options] filename
# -a：仅修改访问时间
# -c：仅修改文件的时间，若该文件不存在则不创建新文件
# -d：后面可接欲修改的日期，也可使用“--date=”的格式
# -m：仅修改mtime
# -t：可接欲修改的时间，格式为[YYMMDDhhmm]
```

### 文件与目录的默认权限与隐藏权限

```shell
[root@yikang ~]# umask
0022<==与后面三个数字表示三组权限，2表示权限被拿掉了2（拿掉了w），若为7则表示无rwx权限
[root@yikang ~]# umask -S
u=rwx,g=rx,o=rx
#设置默认权限
umask number
```

一般新建的文件权限为：-rw-rw-rw，新建的目录为：drwxrwxrwx。

#### 文件隐藏属性chattr, lsattr

```shell
chattr [+-=] [options] file/dir
# +：增加某个参数
# -：删除某个参数
# =：仅有后面接的参数
# S：当进行文件的修改时，该改动会同步地写入磁盘
# a：该文件只能增加数据，而不能删除以及修改数据，只有root才能设置该参数
# c：将自动将文件压缩，当读取时会解压缩
# d：当dump程序执行时，该文件不会被dump备份
# i：不能删除，改名，设置连接也无法写入或添加数据，只有root才能设置
# s：若文件被删除，则完全从磁盘上删除
# u：与s相反，删除后数据还在磁盘上，可找回
```

```shell
lsattr [options] file/dir
# -a：将隐藏文件的属性也显示出来
# -d：若接的是目录，仅列出目录本身的属性而非目录内的文件名
# -R：连同子目录的数据列出来
```

#### 查看文件类型

```shell
file filename
```

### 命令与文件的查询

#### 脚本文件名的查询

```shell
which [-options] command
# -a：由PATH目录中可以找到的命令均列出
```

#### 文件名的查找

find直接查找磁盘，而whereis与locate是根据内置数据库查找的，后两者更快，但有时会查出已删除的文件或是无法查出新创建的文件。

```shell
whereis [options] filename/dir
# -b：只找二进制文件
# -m：只找在说明文件manual路径下的文件
# -s：只找source源文件
# -u：查找不在上述三个选项中的其他特殊文件
```

```shell
locate [options] keyword
# -i：忽略大小写
# -r：后面可接正则表达式的显示方式
```

```shell
find [PATH] [options] [expression] [action]
#与时间相关的参数：-atime, -ctime, -mtime，下面以-mtime为例
# -mtime n：n为数字，表示n天之前的24小时内被更改过的文件
# -mtime +n：列出在n天之前（不包含n天本身）被更改过的文件名
# -mtime -n：列出在n天之内（含n天本身）被更改过的文件名
# -newer file：file为一个存在的文件，列出比file还新的文件名
#与用户或用户组名有关的参数
# -uid n：n为数字，该数字是用户的账号ID，即UID，这个UID是记录在/etc/passwd里面与账号名称相对应的数字。
# -gid n：n为数字，该数字是用户组名的ID，即GID，这个GID是记录在/etc/group中
# -user name：name为用户账号名称
# -group name：name为用户组名
# -nouser：寻找文件的所有者不存在/etc/passwd的人
# -nogroup：寻找文件的所有用户组不存在于/etc/group中的文件。当自行安装软件时，很可能该软件的属性当中没有文件所有者。这时可以使用-nouser与-nogroup查找
#与文件权限及名称有关的参数
# -name filename：查找文件名为filename的文件
# -size [+-] SIZE：查找比SIZE还要大（+）或小（-）的文件。这个SIZE的规格有：
#	c：代表byte，k：代表1024bytes，例如要找比50KB大的文件，“-size +50k”
# -type TYPE：文件类型为TYPE。类型主要有
#	一般正规文件（f）、设备文件（b，c）、目录（d）、连接文件（l）、socket（s）、FIFO（p）等
# -perm mode：查找文件权限“刚好等于mode的权限”的文件。例如找-rwxr--r--的文件时，使用-perm -0744。
# -perm -mode：查找文件权限“必须包含全部mode的权限”的文件。找0744,0755的也会被显示出来
# -perm +mode：查找文件权限“包含任意mode的权限”的文件
```

```shell
find [PATH] [options] [expression] -exec [commend] \;
#注意\;要与前面的字符空一格
#查找结果会放入{}中
find ./ -name '*repor*' -exec ls -l {} \;
```

