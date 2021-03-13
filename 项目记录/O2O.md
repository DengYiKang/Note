# O2O

## 前期准备工作

### 创建项目

#### 创建文件夹并标记

使用Maven的模板webapp创建项目，相比与quick start模板，webapp多了webapp文件夹，并且需要新建src/main/java、src/main/resources、src/test/java、src/test/resources等文件夹，并分别标记成sources root、resources root、test sources root、rest resources root。

创建文件夹并标记后的结构如下：

![](../pic/45.png)

#### WEB-INF文件夹

+ `/WEB-INF/web.xml`：Web应用程序配置文件，描述了 servlet 和其他的应用组件配置及命名规则。
+ `/WEB-INF/classes/`：包含了站点所有用的 class 文件，包括 servlet class 和非servlet class，他们不能包含在 .jar文件中。
+ `/WEB-INF/lib/`：存放web应用需要的各种JAR文件，放置仅在这个应用中要求使用的jar文件,如数据库驱动jar文件。
+ `/WEB-INF/src/`：源码目录，按照包名结构放置各个java文件。
+ `/WEB-INF/database.properties`：数据库配置文件。
+ `/WEB-INF/tags/`：存放了自定义标签文件，该目录并不一定为 tags，可以根据自己的喜好和习惯为自己的标签文件库命名，当使用自定义的标签文件库名称时，在使用标签文件时就必须声明正确的标签文件库路径。例如：当自定义标签文件库名称为 simpleTags 时，在使用 simpleTags 目录下的标签文件时，就必须在 jsp 文件头声明为：`<%@ taglibprefix=“tags” tagdir="/WEB-INF /simpleTags" % >`。
+ `/WEB-INF/jsp/`：jsp 1.2 以下版本的文件存放位置。改目录没有特定的声明，同样，可以根据自己的喜好与习惯来命名。此目录主要存放的是 jsp 1.2 以下版本的文件，为区分 jsp 2.0 文件，通常使用 jsp 命名，当然你也可以命名为 jspOldEdition 。
+ `/WEB-INF/jsp2/`：与 jsp 文件目录相比，该目录下主要存放 Jsp 2.0 以下版本的文件，当然，它也是可以任意命名的，同样为区别 Jsp 1.2以下版本的文件目录，通常才命名为 jsp2。

### 配置Tomcat

#### 安装

查看Tomcat各个版本的兼容性，可以在官网的`Download`列表下的子标签`which version`中找到。

这里使用Tomcat9。

下载Core下的tar.gz，解压到opt目录下。

```shell
cd /opt/tomcat9/bin
sudo gedit ./startup.sh
```

打开后，在最后一行之前添加以下内容：

```shell
export JAVA_HOME=/opt/jdk1.8.0_221
export JRE_HOME=${JAVA_HOME}/jre
export CALSSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
export TOMCAT_HOME=/opt/tomcat9
```

启动tomcat：

```shell
./startup.sh
```

前往[网址](http://localhost:8080/)查看是否正常运行。

#### 配置IDEA

run->Edit Configurations->Add->Tomcat Server->local，选择tomcat的安装路径，配置热部署。

![](../pic/46.png)

配置DeployMent：

![](../pic/47.png)

Application context默认为`/项目名_war_exploded`，可以修改为`/`使得直接使用`/`来访问资源。

##### war和war exploded的区别

是选择`war`还是`war exploded` 这里首先看一下他们两个的区别：

+ `war`模式：将WEB工程以包的形式上传到服务器 ； 
+ `war exploded`模式：将WEB工程以当前文件夹的位置关系上传到服务器；

`war`模式这种可以称之为是发布模式，看名字也知道，这是先打成war包，再发布；

`war exploded`模式是直接把文件夹、jsp页面 、classes等等移到Tomcat 部署文件夹里面，进行加载部署。因此这种方式支持热部署，一般在开发的时候也是用这种方式。

在平时开发的时候，使用热部署的话，应该对Tomcat进行相应的设置，这样的话修改的jsp界面什么的东西才可以及时的显示出来。

## 系统功能模块划分

### 整体

+ 前端展示系统
	+ 头条展示
	+ 店铺类别展示
	+ 区域展示
	+ 店铺
		+ 列表展示
		+ 查询
		+ 详情
	+ 商品
		+ 列表展示
		+ 查询
		+ 详情
+ 超级管理员系统
	+ 头条信息的维护
	+ 店铺类别信息维护
	+ 区域信息维护
	+ 权限验证
	+ 店铺管理
	+ 用户管理

### 实体类解析

#### 实体一览

![](../pic/48.jpg)

#### 实体类设计与数据库表创建

##### 区域

<img src="../pic/49.jpg" style="zoom:80%;" />

```java
public Class Area{
	private Integer areaId;
    private String areaName;
    private Integer priority;
    private Date createTime;
    private Date lastEditTime;
    //getter and setter......
}
```

注意，避免使用基础类型，而是使用对应的类，因为在默认情况下基础类型有值，而引用类型变量为null。

```mysql
create table tb_area
(
    area_id        int(2)       not null auto_increment,
    area_name      varchar(200) not null,
    priority       int(2)       not null default 0,
    create_time    datetime              default null,
    last_edit_time datetime              default null,
    primary key (area_id),
    unique key UK_AREA (area_name)
) engine = InnoDB
  AUTO_INCREMENT = 1
  default charset = utf8
```

##### 用户信息

<img src="../pic/50.jpg" style="zoom:80%;" />

```java
public class PersonInfo {
    private Long userId;
    private String name;
    private String profileImg;
    private String email;
    private String gender;
    private Integer enableStatus;
    //userType:
    //1-顾客
    //2-店家
    //3-超级管理员
    private Integer userType;
    private Date createTime;
    private Date lastEditTime;
    //getter and setter......
}
```

```mysql
create table tb_person_info
(
    user_id        int(10) not null auto_increment,
    name           varchar(32)      default null,
    profile_img    varchar(1024)    default null,
    email          varchar(1024)    default null,
    gender         varchar(2)       default null,
    enable_status  int(2)  not null default 0 comment '0：禁止使用本商城；1：允许使用本商城',
    user_type      int(2)  not null default 1 comment '1：顾客；2：店家；3：超级管理员',
    create_time    datetime         default null,
    last_edit_time datetime         default null,
    primary key (user_id)
) engine = InnoDB
  auto_increment = 1
  default charset = utf8;
```

##### 微信账号和本地账号

![](../pic/51.jpg)

```java
public class WechatAuth {
    private Long wechatAuthId;
    private String openId;
    private Date createTime;
    private PersonInfo personInfo;
}
```

```java
public class LocalAuth {
    private Long localAuthId;
    private String username;
    private String password;
    private Date createTime;
    private Date lastEditTime;
    private PersonInfo personInfo;
}
```

```mysql
create table tb_wechat_auth
(
    wechat_auth_id int(10)       not null auto_increment,
    user_id        int(10)       not null,
    open_id        varchar(1024) not null,
    create_time    datetime default null,
    primary key (wechat_auth_id),
    constraint fk_wechatauth_profile foreign key (user_id) references tb_person_info (user_id)
) engine = InnoDB
  auto_increment = 1
  default charset = utf8;
create table tb_local_auth
(
    local_auth_id  int(10)      not null auto_increment,
    user_id        int(10)      not null,
    username      varchar(128) not null,
    password       varchar(128) not null,
    create_time    datetime default null,
    last_edit_time datetime default null,
    primary key (local_auth_id),
    unique key uk_local_profile (username),
    constraint fk_localauth_profile foreign key (user_id) references tb_person_info (user_id)
) engine = InnoDB
  auto_increment = 1
  default charset = utf8;
# 对open_id建立索引
alter table tb_wechat_auth add unique index (open_id);
```

##### 头条

<img src="../pic/52.jpg" style="zoom:80%;" />

```java
public class HeadLine {
    private Long lineId;
    private String lineName;
    private String lineLink;
    private String lineImg;
    private Integer priority;
    //0:可用；1：不可用
    private Integer enableStatus;
    private Date createTime;
    private Date lastEditTime;
}
```

```mysql
create table tb_head_line
(
    line_id        int(100)      not null auto_increment,
    line_name      varchar(1000)          default null,
    line_link      varchar(2000) not null,
    line_img       varchar(2000) not null,
    priority       int(2)                 default null,
    enable_status  int(2)        not null default 0,
    create_time    datetime               default null,
    last_edit_time datetime               default null,
    primary key (line_id)
) engine = InnoDB
  auto_increment = 1
  default charset = utf8;
```

##### 店铺类别

<img src="../pic/53.jpg" style="zoom:80%;" />