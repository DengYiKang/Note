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

| 命令 | 说明 |
| ---- | ---- |
|git fetch-pack|执行fetch或pull命令时在本地执行此命令，用于从其他版本库获取缺失的对象|
|git receive-pack|执行push命令在远程执行的命令，用于接收推送的数据|
|git send-pack|执行push命令时在本地执行的命令，用于向其他版本库推送数据|
|git upload-archive|执行git archive --remote命令基于远程版本库创建归档时，远程版本库执行此命令传送归档|
|git upload-pack|执行fetch或pull命令时在远程执行此命令，将对象打包、上传|

## 邮件相关命令

| 命令 | 说明 | 命令 | 说明 |
| ---- | ---- | ---- | ---- |
|git imap-send|将补丁通过IMAP发送|git mailinfo|从邮件导出提交说明和补丁|
|git mailsplit|将mbox或Maildir格式邮箱中邮件逐一提取为文件|git request-pull|创建包含提交间差异和执行PULL操作地址的信息|
|git send-email|发送邮件|||

## 协议相关命令

| 命令 | 说明 |
| ---- | ---- |
|git daemon|实现Git协议|
|git http-backend|实现HTTP协议的CGI程序，支持智能HTTP协议|
|git instaweb|即时启动浏览器通过gitweb浏览当前版本库|
|git shell|受限制的shell，提供仅执行Git命令的SSH访问|
|git update-server-info|更新哑协议需要的辅助文件|
|git http-fetch|通过HTTP协议获取版本库|
|git http-push|通过HTTP/DAV协议推送|
|git remote-ext|由Git命令调用，通过外部命令提供扩展协议支持|
|git remote-fd|由Git命令调用，使用文件描述符作为协议接口|
|git remote-ftp|由Git命令调用，提供对FTP协议的支持|
|git remote-ftps|由Git命令调用，提供对FTPS协议的支持|
|git remote-http|由Git命令调用，提供对HTTP协议的支持|
|git remote-https|由Git命令调用，提供对HTTPS协议的支持|
|git remote-testgit|协议扩展示例脚本|

## 版本库转换和交互相关命令

| 命令 | 说明 |
| ---- | ---- |
|git archimport|导入Arch版本库到Git|
|git bundle|提交打包和解包，以便在不同版本库间传递|
|git cvsexportcommit|将Git的一个提交作为一个CVS检出|
|git cvsimport|导入CVS版本库到Git。或者使用cvs2git|
|git cvsserver|Git的CVS协议模拟器，可供CVS命令访问Git版本库|
|git fast-export|将提交到处为git-fast-import格式|
|git fast-import|其他版本库迁移至Git的通用工具|
|git svn|Git作为前端操作Subversion|

## 合并相关的辅助命令

| 命令 | 说明 |
| ---- | ---- |
|git merge-base|供其他脚本调用，找到两个或多个提交最近的公共祖先|
|git merge-file|针对文件的两个不同版本执行三向文件合并|
|git merge-index|对index中的冲突文件调用指定的冲突解决工具|
|git merge-octopus|合并两个以上的分支。参见git merge的octopus合并策略|
|git merge-one-file|由git merge-index调用的标准辅助程序|
|git merge-ours|合并使用本地版本，抛弃他人版本。参见git merge的ours策略|
|git merge-recursive|针对两个分支的三向合并。参见git merge的recursive策略|
|git merge-resolve|针对两个分支的三向合并。参见git merge的resolve策略|
|git merge-tree|显示三向合并结果，不改变暂存区|
|git fmt-merge-msg|供执行合并操作的脚本调用，用于创建一个合并提交说明|
|git rerere|重用所记录的冲突解决方案|

## 杂项

| 命令 | 说明 |
| ---- | ---- |
|git bisect --helper|由git biset命令调用，确认二分查找进度|
|git check-attr|显示某个文件是否设置了某个属性|
|git checkout-index|从暂存区拷贝文件至工作区|
|git cherry|查找没有合并到上游的提交|
|git diff-files|比较暂存区和工作区，相当于git diff --raw|
|git diff-index|比较暂存区和版本库区，相当于git diff --cached --raw|
|git diff-tree|比较两个树对象，相当于git diff --raw A B|
|git difftool --helper|由git difftool命令调用，默认要使用的差异比较工具|
|git get-tar-commit-id|从git archive创建的tar包中提取提交ID|
|git gui --askpass|命令git gui的获取用户口令输入界面|
|git notes|提交评论管理|
|git patch-id|补丁过滤行号和空白字符后生成补丁唯一ID|
|git quiltimport|将Quilt补丁列表应用到当前分支|
|git replace|提交替换|
|git shortlog|对git log的汇总输出，适合于产品发布说明|
|git stripspace|删除空行，供其他脚本调用|
|git submodule|子模组管理|
|git tar-tree|过时命令，请使用git archive|
|git var|显示Git环境变量|
|git web --browse|启动浏览器以查看目录或文件|
|git whatchanged|显示提交历史及每次提交的改动|
|git-mergetool --lib|包含于其他脚本中，提供合并/差异比较工具的选择和执行|
|git-parse-remote|包含于其他脚本中，提供操作远程版本库的函数|
|git-sh-setup|包含于其他脚本中，提供shell编程的函数库|

