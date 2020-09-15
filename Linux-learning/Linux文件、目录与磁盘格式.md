# Linux文件、目录与磁盘格式

### 文件系统的简单操作

#### 磁盘与目录的容量

+ df：列出文件系统的整体磁盘使用量
+ du：评估文件系统的磁盘使用量（常用于评估目录所占容量）

```shell
df [options] [file/dir]
# -h：以人们易阅读的格式显示
# -i：不用硬盘容量，而以inode的数量来显示

[root@yikang /]# df -h /home/
文件系统        容量  已用  可用 已用% 挂载点
/dev/vda1        40G  1.7G   36G    5% /
```

```shell
du [options] file/dir
# -a：列出所有的文件与目录容量，因为默认仅统计目录下面的文件量而已
# -h：易读
# -s：列出总量而已，而不列出每个各别的目录占用容量
# -S：不包括子目录下得到统计
```

### 磁盘的分区、格式化、检验与挂载

```shell
fdisk [options] 设备名称
# -l：输出设备所有的分区内容，仅有fdisk -l时，会把整个系统内能够找到的设备的分区均列出来
```

可以用df来找到磁盘文件名，然后用fdisk来查阅。

```shell
[root@yikang /]# df /
文件系统          1K-块    已用     可用 已用% 挂载点
/dev/vda1      41151808 1781280 37257096    5% /
[root@yikang /]# fdisk /dev/vda1
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。

Device does not contain a recognized partition table
使用磁盘标识符 0x66149749 创建新的 DOS 磁盘标签。

命令(输入 m 获取帮助)：m
命令操作
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition	###
   g   create a new empty GPT partition table
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu
   n   add a new partition	###
   o   create a new empty DOS partition table
   p   print the partition table	###
   q   quit without saving changes	###
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit	###
   x   extra functionality (experts only)
```

#### 磁盘挂载与卸载

```shell
mount -a
mount [-l]
mount [-t 文件系统] [-L Label名] [-o 额外选项] divice dir
# -a：依照配置文件/etc/fstab的数据将所有未挂载的磁盘都挂载上来
# -l：单输入mount会显示目前挂载的信息，加上-l可增列Label的名称
# -t：后接文件系统种类来指定欲挂载的类型，常见的Linux支持类型有ext2、ext3、vfat、reiserfs、iso9660（光盘格式）、nfs、cifs、smbfs（此三种为网络文件系统类型）
# -n：在默认情况下，系统会将实际挂载的情况实时写入/etc/mtab中，以利其他程序的运行。但在一些情况下（如单用户维护模式）为了避免问题，会刻意不写入，此时用该选项
# -L：系统除了用设备名称（如/dev/hdc6）之外，还可以利用文件系统的卷标名称（Label）来挂载。
# -o：后面可以接一些额外参数：
#	ro，rw：挂载文件系统成为只读或可读写
#	async，sync：此文件系统是否使用同步写入（sync）或异步写入（async）。默认为async。
#	auto，noauto：允许分区被以mount -a自动挂载
#	dev，nodev：是否允许分区上可创建设备文件？
#	suid，nosuid：是否允许分区含有suid/sgid的文件格式？
#	exec，noexec：是否允许分区拥有可执行binary文件？
#	user，nouser：是否允许分区让任何用户执行mount？一般来说，mount仅有root可以进行，但下达user参数，则可让一般user也能够对此分区进行mount
#	defaults：默认值为rw，suid，dev，exec，auto，nouser，async
#	remount：重新挂载，这在系统出错，或重新更新参数时，很有用
```

eg:

```bash
#用默认的方式将/dev/hdc6挂载到/mnt/hdc6上
mkdir /mnt/hdc6
mount /dev/hdc6 /mnt/hdc6
#这里未使用-t，在这里Linux自动测试挂载
```

```shell
#查看目前已挂载的文件系统，包含各文件系统的Label名称
mount -l
```

```shell
#安装Linux的CentOS原版光盘挂载
mkdir /media/cdrom
mount -t ios9660 /dev/cdrom /media/cdrom
#也可自动测试挂载
mount /dev/cdrom /media/cdrom
```

```shell
#格式化软盘为Windows与Linux可共同使用的fat格式
mkfs -t vfat /dev/fd0
mkdir /media/floppy
mount -t vfat /dev/fd0 /media/floppy
```

```shell
#挂载U盘，注意U盘不能是NTFS的文件系统
#找出U盘文件名
fdisk -l
mkdir /mnt/flash
mount -t vfat -o iocharset=cp950 /dec/sda1 /mnt/flash
```

```shell
#若根目录出现只读状态，可以重新挂载，加入参数rw与auto
mount -o remount,rw,auto /
```

```shell
#卸载
unmount [options] 设备文件名或挂载点
# -f：强制卸载。可用在类似网络文件系统（NFS）无法读取到的情况下
# -n：不更新/etc/mtab的情况下卸载
```

### 设置开机挂载

#### 开机挂载/etc/fstab以及/etc/mtab

设置开机挂载在/etc/fstab里修改。

但注意系统挂载的一些限制：

+ 根目录/必须挂载，而且要先于其他挂载点被挂载出来
+ 其他挂载点必须为已新建的目录
+ 所有挂载点在同一时间内，只能挂载一次
+ 所有分区在同一时间内，只能挂载一次
+ 如若进行卸载，必须将工作目录移到挂载点之外

```shell
[root@yikang ~]# cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Fri Aug 18 03:51:14 2017
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=59d9ca7b-4f39-4c0c-9334-c56c182076b5 /                       ext4    defaults     1 1
#[Device]	[Mount Point]	[filesystem]	[parameters]	[dump]	[fsck]
#备份用的命令支持dump，0表示不要做dump备份，1表示每天进行dump操作，2表示其他不定日期的dump操作
#开机时是否进行文件系统检验fsck，0表示不要检验，1表示最早检验（一般根目录是1）,2是相对1晚检验（一般其他要检验的文件设置2）
```

如果在/etc/fstab输入了错误的数据导致无法顺利开机，那么应该进入单用户维护模式，可那时的/是只读状态，所以无法修改/etc/fstab，此时可以重新挂载根目录：

```shell
mount -n -o remount,rw /
```

