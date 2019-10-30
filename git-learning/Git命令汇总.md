# Git 命令汇总

## 常用的Git命令

| 命令    | 说明 | 命令    | 说明 |
| ------- | ---- | ------- | ---- |
|git add|添加至缓存区|git add -i|交互式添加|
|git apply|应用补丁|git am|应用邮件格式补丁|
|git annotate|等同于git blame|git archive|文件归档打包|
|git bisect|二分查找|git blame|文件逐行追溯|
|git branch|分支管理|git cat-file|版本库对象研究工具|
|git checkout|检出|git cherry-pick|提交拣选|
|git citool|图形化提交，相当于git gui|git clean|清除工作区未跟踪文件|
|git clone|克隆|git commit|提交|
|git config|查询和修改配置|git describe|通过tag查看提交ID|
|git diff|差异比较|git difftool|调用图形化差异比较工具|
|git fetch|获取远程版本库的提交|git format-patch|创建邮件格式的补丁文件|
|git grep|文件内容搜索|git gui|基于Tcl/Tk的图形化工具，侧重提交等操作|
|git help|帮助|git init|版本库初始化|
|git mergetool|图形化冲突解决|git mv|重命名|
|git rebase|变基|git rebase -i|交互式分支变基|
|git reflog|分支等引用变更记录管理|git remote|远程版本库管理|
|git reset|重置改变HEAD|git rev-parse|将引用转换为哈希值|
|git revert|反转提交|git rm|删除文件|
|git show|显示各种类型对象|git stash|保存和恢复进度|
## 对象库操作相关命令

| 命令 | 说明 | 命令 | 说明 |
| ---- | ---- | ---- | ---- |
|git commit-tree|从树对象创建提交|git hash-object|从标准输入或文件计算哈希值或创建对象|
|git ls-files|显示工作区和暂存区文件|git ls-tree|显示树对象包含的文件|
|git mktag|读取标准输入创建一个tag对象|git mktree|读取标准输入创建一个树对象|
|git read-tree|读取树对象到暂存区|git update-index|工作区内容注册到暂存区及暂存区管理|
|git unpack-file|创建临时文件包含指定blob的内容|git write-tree|从暂存区创建一个树对象|

## 引用操作相关命令

| 命令 | 说明 | 命令 | 说明 |
| ---- | ---- | ---- | ---- |
|git check-ref-format|检查引用名称是否符合规范|git for-each-ref|引用迭代器，用于shell编程|
|git ls-remote|显示远程版本库的引用|git name-rev|将提交ID显示为名称|
|git rev-list|显示版本范围|git show-branch|显示分支列表及拓扑关系|
|git show-ref|显示本地引用|git symbolic-ref|显示或者设置符号引用|
|git update-ref|更新引用的指向|git verify-tag|校验GPG签名的Tag|

## 版本库管理相关命令

| 命令 | 说明 | 命令 | 说明 |
| ---- | ---- | ---- | ---- |
|git count-objects|显示松散对象的数量和磁盘占用|git filter-branch|版本库重构|
|git fsck|对象库完整性检查|git gc|版本库存储优化|
|git index-pack|从打包文件创建对应的索引文件|git pack-objects|从标准输入读入对象ID，打包到文件|
|git pack-redundant|查找多余的pack文件|git pack-refs|将引用打包到.git/packed-refs文件中|
|git prune|从对象库中删除过期对象|git prune-packed|将已经打包的松散对象删除|
|git relink|为本地版本库中相同的对象建立硬连接|git repack|将库中未打包的松散对象打包|
|git show-index|读取包的索引文件，显示打包文件中的内容|git unpack-objects|从打包文件释放文件|
|git verify-pack|校验对象库打包文件|||

## 数据传输相关命令


