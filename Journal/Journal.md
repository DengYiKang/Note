## Journal

## 04.09

maven 附加-X可以debug。使用mvn -X archetype:generate命令debug时发现阿里镜像上没有archetype-catelog.xml文件，本地也没有。

IDEA配置maven，修改maven源为阿里云后，构建项目时缺少archetype-category.xml（阿里镜像没有？）。只需把该文件下载下来放到.m2/repository，然后在IDEA的maven的runner设置vm-options为-DarchetypeCatalog=local。

配置了.IntelliJIdea2019.3/system/Maven/Indices/UserArchetypes.xml文件，在IDEA的archetype选择界面并没有显示？是格式不对？建议还是使用maven构建比较好，然后用IDEA打开。

