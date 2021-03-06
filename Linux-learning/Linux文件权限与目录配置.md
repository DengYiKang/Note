## Linux文件权限与目录配置

### 文件属性

```bash
[root@yikang ~]# ls -al
总用量 56
dr-xr-x---.  5 root root 4096 9月  12 20:37 .
dr-xr-xr-x. 18 root root 4096 9月  11 15:13 ..
-rw-------   1 root root  424 9月  12 22:49 .bash_history
-rw-r--r--.  1 root root   18 12月 29 2013 .bash_logout
-rw-r--r--.  1 root root  176 12月 29 2013 .bash_profile
-rw-r--r--.  1 root root  176 12月 29 2013 .bashrc
drwx------   3 root root 4096 8月  18 2017 .cache
-rw-r--r--.  1 root root  100 12月 29 2013 .cshrc
-rw-------   1 root root   39 9月  12 20:37 .lesshst
drwxr-xr-x   2 root root 4096 8月  18 2017 .pip
-rw-r--r--   1 root root   64 8月  18 2017 .pydistutils.cfg
drwx------   2 root root 4096 9月  11 15:06 .ssh
-rw-r--r--.  1 root root  129 12月 29 2013 .tcshrc
-rw-------   1 root root  705 9月  12 20:16 .viminfo
[权限][连接][所有者][用户组][文件容量][修改日期][文件名]
```

第一列表示文件的类型与权限，共有十个字符。

+ 第一个字符代表这个文件的类型：
  + [d]表示目录
  + [-]表示文件
  + [l]表示连接文件(linkfile)
  + [b]表示设备文件里面的可供存储的接口设备
  + [c]表示设备文件里面的串行端口设备，例如键盘，鼠标等。

+ 接下来的字符中，以3个为一组，均为“r-w-x”的3个参数的组合。其中[r]表示可读，[w]代表可写，[x]代表可执行。
  + 第一组为文件所有者的权限
  + 第二组为同用户组的权限
  + 第三组为其他非本用户组的权限

第二列表示有多少文件名连接到此结点。

第三列表示这个文件的所有者账号。

第四列表示这个文件的所属用户组。

第五列表示这个文件的容量大小，默认单位为B。

第六列表示这个文件的创建文件日期或者是最近的修改日期。

>注意，若权限为r--的形式，且其为目录，则对应用户无法进入此目录！

### 如何改变文件属性与权限

#### 改变文件属性

```bash
#改变文件所属用户组，用户组信息一般存在/etc/group中
chgrp [-R] dirname/filename
#改变文件所有者，用户账号一般存在/etc/passwd中
chown [-R] 账号名称 文件或目录
#还可以同时更改用户组
chown [-R] 账号名称:组名 文件或目录
```

#### 改变文件权限

使用chmod时，用数字来代表各个权限，各权限的分数对照表如下：

| 权限 | 数字 |
| ---- | ---- |
| r    | 4    |
|w|2|
|x|1|

每种身份(owner, group, others)各自的三个权限（r, w, x）分数是需要累加的，例如当权限为[-rwxrwx---]，分数为：

+ owner=rwx=4+2+1=7
+ group=rwx=4+2+1=7
+ others=---=0+0+0=0

```shell
#xyz即分数
chmod [-R] xyz dir/file
```

> 例如编辑一个shell的批处理文件后，它的权限通常是"-rw-rw-r-"，也就是664，如果要将该文件变成可执行文件，且不允许其他人修改，则需要"-rwxr-xr-x"这样的权限，也就是755。

> 如果你不希望文件被其他人看到，那么可以设置成"-rwxr-----"，也就是740.

还有另一种改变权限的方式：

|       |            |           |         |            |
| ----- | ---------- | --------- | ------- | ---------- |
| chmod | u, g, o, a | +,  -,  = | r, w, x | 文件或目录 |

其中，u，g，o，a分别表示user、group、others、all四种身份。+，-，=分别表示加入、除去、设置。

```shell
#更改file权限为user rwx， g和o为r-x
chmod u=rwx, go=rx file
#使每个人均有写入的权限
chmod a+w file
#去除掉每个人的可执行权限
chmod a-x file
```

> 注意，Windows下的文件是根据扩展名来判断是否可执行，而Linux下的文件有是否具有x的权限决定的！

> 文件（不是目录）的w权限具有写入、编辑、新增、修改的权限，但不含删除的权限！

> 目录的x权限表示用户能否进入该目录，也无法执行该目录下的所有命令，没有x权限但有r权限还是查询目录下的列表的（但是详细信息读不到）。开放目录给他人浏览时，至少要给予r与x的权限。

> 一个在主文件夹下的无rwx权限的文件，虽然无rwx权限，但因为目录是自己的，所有有删除该文件的权限。

### 目录标准配置：FHS

|          | 可分享的                                            | 不可分享的                                 |
| -------- | --------------------------------------------------- | ------------------------------------------ |
| 不变的   | /usr(软件放置处)<br>/opt(第三方软件)                | /etc(配置文件)<br>/boot(开机与内核文件)    |
| 可变动的 | /var/mail(用户邮件信箱)<br> /var/spool/news(新闻组) | /var/run(程序相关)<br> /var/lock(程序相关) |

+ 可分享的：可以分享给其他系统挂载使用的目录，是能分享给网络上其他主机挂载用的目录。
+ 不变的：数据不经常变动。

FHS建议根目录（/）下的子目录：

| 目录        | 应放置文件内容                                               |
| ----------- | ------------------------------------------------------------ |
| /bin        | 在单用户维护模式下还能被操作的命令，通常可以被root与一般账号使用 |
| /boot       | 开机会使用的文件，包括内核文件、开机菜单和开机所需配置文件等。Linux kernel常用文件名为vmlinuz |
| /dev        | 设备文件                                                     |
| /etc        | 系统的主要配置文件，FHS建议不要放置可执行文件。<br>+ /etc/init.d: 所有服务的默认启动脚本。例如启动iptables：/etc/init.d/iptables start<br>+ /etc/xinetd.d: super daemon管理的各项服务的配置文件目录<br>+ /etc/X11/: 与X Windows有关的配置文件 |
| /home       | 用户主文件夹                                                 |
| /lib        | 开机时用到的函=函数库，以及在/bin或/sbin下的命令会调用的函数库 |
| /media      | 可删除的设备，包括软盘、光盘、DVD等                          |
| /mnt        | 若想暂时挂载某些额外设备，建议放置在该目录                   |
| /opt        | 第三方软件                                                   |
| /root       | 系统管理员的主文件夹                                         |
| /sbin       | 开机过程中需要的命令，包括了开机、修复、还原系统等。只有某些服务器软件程序，一般放置到/usr/sbin中。至于本机自行安装的软件所产生的系统执行文件，则放置到/usr/local/sbin/中 |
| /srv        | 一些网络服务启动之后，这些服务所需取用的数据目录             |
| /tmp        | 一般用户或正在执行的程序暂时存放文件的地方。建议开机时清理   |
| /lost+found | 当文件系统发生错误时，将一些丢失的片段放置到这个目录下       |
| /proc       | 虚拟文件系统。它放置的数据是在内存中的，本身不占空间         |
| /sys        | 虚拟文件系统。记录与内核相关的信息，包括目前已加载的内核模块与内核检测到的硬件设备信息等 |

> 注意因为开机过程仅有根目录会被挂载，其他分区是开机完成之后。因此，与开机相关的目录必须放在根目录下：
>
> + /etc: 配置文件
> + /bin: 重要执行文件
> + /dev: 所需要的设备文件
> + /lib: 执行文件所需的函数库与内核所需的模块
> + /sbin: 重要的系统执行文件

/usr下的目录（usr是UNIX Software Resource的缩写）：

| 目录          | 应放置的文件内容                                             |
| ------------- | ------------------------------------------------------------ |
| /usr/X11R6/   | 为X Windows重要数据放置的目录                                |
| /usr/bin/     | 绝大部分的用户可使用命令（注意区别/bin是与开机有关）         |
| /usr/include/ | C/C++等程序语言header与include放置处                         |
| /usr/lib/     | 包含各种应用软件的函数库、目标文件，以及不被一般用户惯用的执行文件或脚本。 |
| /usr/local/   | 系统管理员在本机自行安装的软件                               |
| /usr/sbin/    | 非系统正常运行所需要的系统命令                               |
| /usr/share/   | 放置共享文件                                                 |
| /usr/src/     | 源码                                                         |

/var下的目录：

| 目录        | 应放置文件内容                                               |
| ----------- | ------------------------------------------------------------ |
| /var/cache/ | 应用程序运行过程中产生的暂存文件                             |
| /var/lib/   | 程序本身执行过程中，需要使用的数据文件放置的目录。例如，MySQL的数据库放置到/var/lib/mysql |
| /var/lock/  | 锁                                                           |
| /var/log/   | 登录文件放置的目录                                           |
| /var/mail/  | 放置个人电子邮件信箱的目录                                   |
| /var/run/   | 某些程序或服务启动后，会将它们的PID放置在该目录下            |
| /var/spool/ | 队列数据，即排队等待其他程序使用的数据                       |

### CentOS的查看

```shell
[root@yikang ~]# uname -r
3.10.0-514.26.2.el7.x86_64<==内核版本
[root@yikang ~]# lsb_release -a
LSB Version:	:core-4.1-amd64:core-4.1-noarch
Distributor ID:	CentOS
Description:	CentOS Linux release 7.3.1611 (Core) 
Release:	7.3.1611
Codename:	Core
```

