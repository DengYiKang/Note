# Git 检出

## git checkout

```bash
用法一：git checkout [-q] [<commit>] [--] [paths]...
用法二：git checkout [<branch>]
用法三：git checkout [-m] [[-b|--orphan]<new_branch>] [<start_point>]
```

第一种用法的<commit>是可选项，如果省略则相当于从暂存区(index)进行检出。这和reset重置命令大不相同：重置的默认值是HEAD，而检出的默认值是暂存区。因此重置一般用于重置暂存区（除非使用--hard，否则不重置工作区），而检出命令主要是覆盖工作区（如果<commit>不省略，也会替换暂存区的相应文件）。

第一种用法不会改变HEAD头指针，主要用于指定版本的文件覆盖工作区中对应的文件。如果省略<commit>，会拿暂存区的文件覆盖工作区的文件，否则用指定提交中的文件**覆盖暂存区和工作区**的对应文件。

第二种用法会改变HEAD头指针。之所以后面的参数写作<branch>，是因为只有HEAD切换到一个分支才可以对提交进行跟踪，否则仍然会进入“分离头指针”的状态。在“分离头指针”状态下的提交不能被引用关联到而可能会丢失。所以用法二最主要的作用就是切换到分支。如果省略<branch>则相当于对工作区进行状态检查。

第三种用法主要是创建和切换到新的分支（**<new_branch>**），新的分支从<s**tart_point**>指定的提交开始创建。

示例：

```bash
git checkout branch				#检出branch分支
git checkout					#汇总显示暂存区与HEAD的差异
git checkout HEAD				#同上
git checkout -- <filename>		#用暂存区的文件覆盖工作区的对应文件
git checkout <branch> -- <filename>
#HEAD的指向不变，将branch指向的提交中的对应文件替换暂存区和工作区中相应文件。
git checkout --. or git checkout .
#取消所有本地的修改（相对于暂存区）。相当于将暂存区的所有文件直接覆盖本地文件。
```

