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