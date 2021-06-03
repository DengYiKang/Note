# pytorch

## conda安装

### 安装

去清华源下载x86_64的sh文件，在命令行中执行，进入安装流程。

conda默认安装在`/home/$USER/`下，注意在安装过程中会询问是否初始化（添加环境变量），选择no。安装成功后添加环境变量，将conda目录下的bin目录添加到环境变量中。

对于镜像，清华源的镜像不太管用，因此使用了默认源，如果已经添加了乱七八糟的源，可以使用以下命令重置：

```shell
conda config --remove-key channels
```

可以使用以下命令查看添加的源：

```shell
conda config --show channels
```

也可以查看`~/.condarc`文件。

