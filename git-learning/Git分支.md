# Git 分支

## 分支命令概述

```bash
用法1：git branch
用法2：git branch <brname>
用法3：git branch <brname> <start_point>
用法4：git branch -d <brname>
用法5：git branch -D <brname>
用法6：git branch -m <oldbr> <newbr>
用法7：git branch -M <oldbr> <newbr>
```

说明：

+ 用法1用于显示本地分支列表。

+ 用法2和用法3用于创建分支。

  用法2基于当前HEAD指向的提交创建分支。

  用法3基于提交<start_point>创建分支。

+ 用法4和用法5用于删除分支。

  用法4在删除分支前会检查所要删除的分支是否已经合并到其他分支，否则拒绝删除。

  用法5强制删除。

+ 用法6和用法7用于重命名分支。

  如果版本库中已经存在同名分支，用法6将拒绝执行，而用法7会强制执行。

```bash
git push <remote_name> -d <brname>
```

