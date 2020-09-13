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
ls [option] [dir]
# -a:全部文件，包含隐藏文件(开头为.)
# -d:仅列出目录本身，不列出目录内的文件数据
# -h:易读方式
# -l:长数据串，包含文件的属性与权限等
# -S:以文件容量大小排序
# -t:以时间排序
```

