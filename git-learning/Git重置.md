# Git 重置

[TOC]



## git reflog

```bash
git reflog show <branch>			#注意最新改变放在最前面显示
```

## git reset

```bash
用法一：git reset [-q] [<commit>] [--] <paths>...
用法二：git reset [--soft|--mixed|--hard|--merge|--keep] [-q] [<commit>]
```

上面两个用法中，<commit>是可选项，可以使用引用或提交**ID**，若省略则默认**HEAD**。

第一种用法不会重置引用，不会改变工作区，而是用指定提交状态(<commit>)下的文件(<paths>)替换掉暂存区中的文件。例如：

```bash
git reset HEAD <path>		#取消之前执行的git add <paths>命令
```

第二种用法则会重置引用。根据不同选项，有：

+ **--hard**
  + 替换引用的指向。引用指向新的提交**ID**
  + 替换暂存区。替换后，暂存区的内容和引用指向的目录树一致
  + 替换工作区。替换后，工作区的内容和暂存区一致，也和**HEAD**所指向的目录树的内容相同
+ **--mixed**（缺省为**mixed**）
  + 更改引用的指向以及重置暂存区，但不改变工作区
+ **--soft**
  + 只更改引用的指向以及重置暂存区，但是不改变工作区