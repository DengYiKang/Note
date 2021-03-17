# O2O

## 前期准备工作

### 创建项目

#### 创建文件夹并标记

使用Maven的模板webapp创建项目，相比与quick start模板，webapp多了webapp文件夹，并且需要新建src/main/java、src/main/resources、src/test/java、src/test/resources等文件夹，并分别标记成sources root、resources root、test sources root、rest resources root。

创建文件夹并标记后的结构如下：

![](../pic/45.png)

之后还需要创建一些文件夹，最终结构如下：

![](../pic/61.png)

#### WEB-INF文件夹

+ `/WEB-INF/web.xml`：Web应用程序配置文件，描述了 servlet 和其他的应用组件配置及命名规则。
+ `/WEB-INF/classes/`：包含了站点所有用的 class 文件，包括 servlet class 和非servlet class，他们不能包含在 .jar文件中。
+ `/WEB-INF/lib/`：存放web应用需要的各种JAR文件，放置仅在这个应用中要求使用的jar文件,如数据库驱动jar文件。
+ `/WEB-INF/src/`：源码目录，按照包名结构放置各个java文件。
+ `/WEB-INF/database.properties`：数据库配置文件。
+ `/WEB-INF/tags/`：存放了自定义标签文件，该目录并不一定为 tags，可以根据自己的喜好和习惯为自己的标签文件库命名，当使用自定义的标签文件库名称时，在使用标签文件时就必须声明正确的标签文件库路径。例如：当自定义标签文件库名称为 simpleTags 时，在使用 simpleTags 目录下的标签文件时，就必须在 jsp 文件头声明为：`<%@ taglibprefix=“tags” tagdir="/WEB-INF /simpleTags" % >`。
+ `/WEB-INF/jsp/`：jsp 1.2 以下版本的文件存放位置。改目录没有特定的声明，同样，可以根据自己的喜好与习惯来命名。此目录主要存放的是 jsp 1.2 以下版本的文件，为区分 jsp 2.0 文件，通常使用 jsp 命名，当然你也可以命名为 jspOldEdition 。
+ `/WEB-INF/jsp2/`：与 jsp 文件目录相比，该目录下主要存放 Jsp 2.0 以下版本的文件，当然，它也是可以任意命名的，同样为区别 Jsp 1.2以下版本的文件目录，通常才命名为 jsp2。

> 外界不能通过url的方式访问WEB-INF文件夹，因此一般把html等资源放在WEB-INF文件夹下。

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

```java
public class ShopCategory {
    private Long shopCategoryId;
    private String shopCategoryName;
    private String shopCategoryDesc;
    private String shopCategoryImg;
    private Integer priority;
    private Date createTime;
    private Date lastEditTime;
    private ShopCategory parent;
}
```

##### 店铺

<img src="../pic/54.jpg" style="zoom:80%;" />

```java
public class Shop {
    private Long shopId;
    private String shopName;
    private String shopDesc;
    private String shopAddr;
    private String phone;
    private String shopImg;
    private Integer priority;
    private Date createTime;
    private Date lastEditTime;
    //-1-不可用，0-审核中，1-可用
    private Integer enableStatus;
    //超级管理员给店家的提醒
    private String advice;
    private Area area;
    private PersonInfo owner;
    private ShopCategory shopCategory;
}
```

```mysql
create table tb_shop
(
    shop_id          int(10)      not null auto_increment,
    owner_id         int(10)      not null comment '店铺创建人',
    area_id          int(5)                default null,
    shop_category_id int(11)               default null,
    shop_name        varchar(256) not null,
    shop_desc        varchar(1024)         default null,
    shop_addr        varchar(200)          default null,
    phone            varchar(128)          default null,
    shop_img         varchar(1024)         default null,
    priority         int(3)                default 0,
    create_time      datetime              default null,
    last_edit_time   datetime              default null,
    enable_status    int(2)       not null default 0,
    advice           varchar(255)          default null,
    primary key (shop_id),
    constraint fk_shop_area foreign key (area_id) references tb_area (area_id),
    constraint fk_shop_profile foreign key (owner_id) references tb_person_info (user_id),
    constraint fk_shop_shopcate foreign key (shop_category_id) references tb_shop_category (shop_category_id)
) engine = InnoDB
  auto_increment = 1
  default charset = utf8;
```

##### 商品类别

<img src="../pic/55.jpg" style="zoom:80%;" />

```java
public class ProductCategory {
    private Long productCategoryId;
    private Long shopId;
    private String productCategoryName;
    private Integer priority;
    private Date createTime;
}
```

```mysql
create table tb_product_category
(
    product_category_id   int(11)      not null auto_increment,
    product_category_name varchar(100) not null,
    priority              int(2)                default 0,
    create_time           datetime              default null,
    shop_id               int(20)      not null default 0,
    primary key (product_category_id),
    constraint fk_procate_shop foreign key (shop_id) references tb_shop (shop_id)
) engine = InnoDB
  auto_increment = 1
  default charset = utf8;
```

##### 详情图片

<img src="../pic/56.jpg" style="zoom:80%;" />

```java
public class ProductImg {
    private Long productImgId;
    private String imgAddr;
    private String imgDesc;
    private Integer priority;
    private Date createTime;
    private Long productId;
}
```

```mysql
# 注意这张表要在tb_product表之后创建
create table tb_product_img
(
    product_img_id int(20)       not null auto_increment,
    img_addr       varchar(2000) not null,
    img_desc       varchar(2000) default null,
    priority       int(2)        default 0,
    create_time    datetime      default null,
    product_id     int(20)       default null,
    primary key (product_img_id),
    constraint fk_proimg_product foreign key (product_id) references tb_product (product_id)
) engine = InnoDB
  auto_increment = 1
  default charset = utf8;
```

因为商品类需要引用详情图片类，因此对于类的定义，详情图片类在商品类之前。

因为详情图片表需要引用商品表，因此对于建表，商品表在详情图片表之前。

##### 商品

<img src="../pic/57.jpg" style="zoom:80%;" />

```java
public class Product {
    private Long productId;
    private String productName;
    private String productDesc;
    //简略图
    private String imgAddr;
    private String normalPrice;
    private String promotionPrice;
    private Integer priority;
    private Date createTime;
    private Date lastEditTime;
    //0-下架，1-在前端展示系统展示
    private Integer enableStatus;
    private List<ProductImg> productImgList;
    private ProductCategory productCategory;
    private Shop shop;
}
```

```mysql
create table tb_product
(
    product_id          int(100)     not null auto_increment,
    product_name        varchar(100) not null,
    product_desc        varchar(2000)         default null,
    img_addr            varchar(2000)         default '',
    normal_price        varchar(100)          default null,
    promotion_price     varchar(100)          default null,
    priority            int(2)       not null default 0,
    create_time         datetime              default null,
    last_edit_time      datetime              default null,
    enable_status       int(2)       not null default 0,
    product_category_id int(11)               default null,
    shop_id             int(20)      not null default 0,
    primary key (product_id),
    constraint fk_product_procate foreign key (product_category_id) references tb_product_category (product_category_id),
    constraint fk_product_shop foreign key (shop_id) references tb_shop (shop_id)
) engine = InnoDB
  auto_increment = 1
  default charset = utf8;
```

#### 用户信息关联

<img src="../pic/58.jpg" style="zoom:80%;" />

#### 店铺信息关联

<img src="../pic/59.jpg" style="zoom:80%;" />

#### 商品信息关联

<img src="../pic/60.jpg" style="zoom:80%;" />

## 基本配置

### 目录结构

spring、mybatis等配置文件通常放在src/main/resources目录下。

![](../pic/62.png)

### jdbc.properties

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/o2o?useUnicode=true&characterEncoding=utf8&serverTimezone=UTC
jdbc.username=root
jdbc.password=159753
```

### mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 配置全局属性 -->
    <settings>
        <!-- 使用jdbc的getGeneratedKeys获取数据库自增主键值 -->
        <setting name="useGeneratedKeys" value="true" />

        <!-- 使用列标签替换列别名 默认:true -->
        <setting name="useColumnLabel" value="true" />

        <!-- 开启驼峰命名转换:Table{create_time} -> Entity{createTime} -->
        <setting name="mapUnderscoreToCamelCase" value="true" />
    </settings>
</configuration>
```

### spring-dao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 配置整合mybatis过程 -->
    <!-- 1.配置数据库相关参数properties的属性：${url} -->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!-- 2.数据库连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <!--配置连接池属性-->
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <!-- c3p0连接池的私有属性 -->
        <property name="maxPoolSize" value="30"/>
        <property name="minPoolSize" value="10"/>
        <!-- 关闭连接后不自动commit -->
        <property name="autoCommitOnClose" value="false"/>
        <!-- 获取连接超时时间 -->
        <property name="checkoutTimeout" value="10000"/>
        <!-- 当获取连接失败重试次数 -->
        <property name="acquireRetryAttempts" value="2"/>
    </bean>

    <!-- 3.配置SqlSessionFactory对象 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 注入数据库连接池 -->
        <property name="dataSource" ref="dataSource"/>
        <!-- 配置MyBaties全局配置文件:mybatis-config.xml -->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <!-- 扫描entity包 使用别名 -->
        <property name="typeAliasesPackage" value="com.yikang.o2o.entity"/>
        <!-- 扫描sql配置文件:mapper需要的xml文件 -->
        <property name="mapperLocations" value="classpath:mapper/*.xml"/>
    </bean>

    <!-- 4.配置扫描Dao接口包，动态实现Dao接口，注入到spring容器中 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 注入sqlSessionFactory -->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <!-- 给出需要扫描Dao接口包 -->
        <property name="basePackage" value="com.yikang.o2o.dao"/>
    </bean>
</beans>
```

### spring-service.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx.xsd">
    <!-- 扫描service包下所有使用注解的类型 -->
    <context:component-scan base-package="com.yikang.o2o.service" />

    <!-- 配置事务管理器 -->
    <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 注入数据库连接池 -->
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!-- 配置基于注解的声明式事务 -->
    <tx:annotation-driven transaction-manager="transactionManager" />
</beans>
```

### spring-web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/mvc
    http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd">
    <!-- 配置SpringMVC -->
    <!-- 1.开启SpringMVC注解模式 -->
    <mvc:annotation-driven />

    <!-- 2.静态资源默认servlet配置 (1)加入对静态资源的处理：js,gif,png (2)允许使用"/"做整体映射 -->
    <mvc:resources mapping="/resources/**" location="/resources/" />
    <mvc:default-servlet-handler />

    <!-- 3.定义视图解析器 -->
    <bean id="viewResolver"
          class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--利用WEB-INF的安全性，在外界不能直接输入/WEB-INF/...url来访问文件的，
        而程序内部是可以通过WEB-INF去访问的，这时可以使用spring的dispatcher servlet来转向，
        我们先去请求一个controller，由controller分发到WEB-INF下面的页面就可以了-->
        <property name="prefix" value="/WEB-INF/html/"></property>
        <property name="suffix" value=".html"></property>
    </bean>

    <!-- 4.扫描web相关(controller)的bean -->
    <context:component-scan base-package="com.yikang.o2o.web" />
</beans>
```

### web.xml

使上述配置文件被识别。

```xml
<!DOCTYPE web-app PUBLIC
        "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
        "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
    <display-name>Archetype Created Web Application</display-name>
    <servlet>
        <servlet-name>spring-dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/spring-*.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>spring-dispatcher</servlet-name>
        <!--默认匹配所有的请求-->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

### pom.xml

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <spring.version>5.1.8.RELEASE</spring.version>
</properties>

<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <!-- https://mvnrepository.com/artifact/ch.qos.logback/logback-classic -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.3</version>
    </dependency>
    <!-- Spring -->
    <!-- 1)包含Spring 框架基本的核心工具类。Spring 其它组件要都要使用到这个包里的类，是其它组件的基本核心 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- 2)这个jar 文件是所有应用都要用到的，它包含访问配置文件、创建和管理bean 以及进行Inversion of Control
        / Dependency Injection（IoC/DI）操作相关的所有类。如果应用只需基本的IoC/DI 支持，引入spring-core.jar
        及spring-beans.jar 文件就可以了。 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- 3)这个jar 文件为Spring 核心提供了大量扩展。可以找到使用Spring ApplicationContext特性时所需的全部类，JDNI
        所需的全部类，instrumentation组件以及校验Validation 方面的相关类。 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- 4) 这个jar 文件包含对Spring 对JDBC 数据访问进行封装的所有类。 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- 5) 为JDBC、Hibernate、JDO、JPA等提供的一致的声明式和编程式事务管理。 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- 6)Spring web 包含Web应用开发时，用到Spring框架时所需的核心类，包括自动载入WebApplicationContext特性的类、Struts与JSF集成类、文件上传的支持类、Filter类和大量工具辅助类。 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- 7)包含SpringMVC框架相关的所有类。 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- 8)Spring test 对JUNIT等测试框架的简单封装 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring.version}</version>
        <scope>test</scope>
    </dependency>
    <!-- Servlet web -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.1</version>
    </dependency>
    <!-- json解析 -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.9</version>
    </dependency>
    <!-- Map工具类 对标准java Collection的扩展 spring-core.jar需commons-collections.jar -->
    <dependency>
        <groupId>commons-collections</groupId>
        <artifactId>commons-collections</artifactId>
        <version>3.2.2</version>
    </dependency>
    <!-- DAO: MyBatis -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.1</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.1</version>
    </dependency>
    <!-- 数据库 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.16</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/com.mchange/c3p0 -->
    <dependency>
        <groupId>com.mchange</groupId>
        <artifactId>c3p0</artifactId>
        <version>0.9.5.4</version>
    </dependency>

    <!-- 图片处理 -->
    <!-- https://mvnrepository.com/artifact/net.coobird/thumbnailator -->
    <dependency>
        <groupId>net.coobird</groupId>
        <artifactId>thumbnailator</artifactId>
        <version>0.4.8</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/com.github.penggle/kaptcha -->
    <dependency>
        <groupId>com.github.penggle</groupId>
        <artifactId>kaptcha</artifactId>
        <version>2.3.2</version>
    </dependency>
    <dependency>
        <groupId>commons-fileupload</groupId>
        <artifactId>commons-fileupload</artifactId>
        <version>1.3.2</version>
    </dependency>
    <!-- redis客户端:Jedis -->
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>2.9.0</version>
    </dependency>

</dependencies>
```

### 验证

AreaDao接口：

```java
public interface AreaDao {
    /**
     * 列出区域列表
     * @return areaList
     */
    List<Area> queryAreas();
}
```

AreaDao的实现AreaDao.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.yikang.o2o.dao.AreaDao">
    <select id="queryAreas" resultType="com.yikang.o2o.entity.Area">
      SELECT area_id, area_name,
      priority, create_time, last_edit_time
      FROM tb_area
      ORDER BY priority
      DESC
   </select>
</mapper>
```

新建文件夹：

![](../pic/63.png)

#### BaseTest.java

```java
/**
 * 配置spring和junit整合，junit启动时加载springIOC容器
 */
@RunWith(SpringJUnit4ClassRunner.class)
//告诉junit spring配置文件的位置
@ContextConfiguration({"classpath:spring/spring-dao.xml"})
public class BaseTest {

}
```

#### AreaDaoTest.java

```java
public class AreaDaoTest extends BaseTest {
    @Autowired
    private AreaDao areaDao;

    @Test
    public void testQueryArea() {
        List<Area> areaList = areaDao.queryAreas();
        assertEquals(2, areaList.size());
    }
}
```

## LogBack日志配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--每分钟扫描以下配置-->
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!-- 定义参数常量 -->
    <!-- TRACE<DEBUG<INFO<WARN<ERROR -->
    <!-- logger.trace("msg") logger.debug... -->
    <property name="log.level" value="debug" />
    <property name="log.maxHistory" value="30" />
    <property name="log.filePath" value="${catalina.base}/logs/webapps" />
    <property name="log.pattern"
              value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n" />
    <!-- 控制台设置 -->
    <appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
    </appender>
    <!-- DEBUG -->
    <appender name="debugAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 文件路径 -->
        <file>${log.filePath}/debug.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 文件名称 -->
            <fileNamePattern>${log.filePath}/debug/debug.%d{yyyy-MM-dd}.log.gz
            </fileNamePattern>
            <!-- 文件最大保存历史数量 -->
            <maxHistory>${log.maxHistory}</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>DEBUG</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!-- INFO -->
    <appender name="infoAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 文件路径 -->
        <file>${log.filePath}/info.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 文件名称 -->
            <fileNamePattern>${log.filePath}/info/info.%d{yyyy-MM-dd}.log.gz
            </fileNamePattern>
            <!-- 文件最大保存历史数量 -->
            <maxHistory>${log.maxHistory}</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!-- ERROR -->
    <appender name="errorAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 文件路径 -->
        <file>${log.filePath}/erorr.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 文件名称 -->
            <fileNamePattern>${log.filePath}/error/error.%d{yyyy-MM-dd}.log.gz
            </fileNamePattern>
            <!-- 文件最大保存历史数量 -->
            <maxHistory>${log.maxHistory}</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!--name表示logger关注的是哪个package-->
    <!--additivity=true表示继承root里的appender，即也会包含控制台的输出-->
    <logger name="com.yikang.o2o" level="${log.level}" additivity="true">
        <appender-ref ref="debugAppender"/>
        <appender-ref ref="infoAppender"/>
        <appender-ref ref="errorAppender"/>
    </logger>
    <root level="info">
        <appender-ref ref="consoleAppender"/>
    </root>
</configuration>
```

其中，配置log.filePath时用到了`${catalina.base}`这个变量。

+ `catalina.home`：Tomcat 安装目录，一般是用来查找库 jar 的。
+ `catalina.base`：服务器配置目录。

在IDEA里启动Tomcat时会发现控制台输出以下信息：

![](../pic/65.png)

会发现`catalina.base`是IDEA下的当前项目的工作环境里的目录。

## Thumbnailator图片处理和封装Util

### ImageUtil

```java
public class ImageUtil {
    //获取resources目录路径
    private static String basePath = Thread.currentThread().getContextClassLoader().getResource("").getPath();
    private static final SimpleDateFormat sDateFormat = new SimpleDateFormat("yyyyMMddHHmmss");
    private static final Random r = new Random();
    private static final Logger logger= LoggerFactory.getLogger(ImageUtil.class);

    /**
     * 将CommonsMultipartFile转换成File类
     *
     * @param cFile
     * @return
     */
    public static File transferCommonsMultipartFileToFile(CommonsMultipartFile cFile) {
        File newFile = new File(cFile.getOriginalFilename());
        try {
            cFile.transferTo(newFile);
        } catch (IllegalStateException e) {
            logger.error(e.toString());
            e.printStackTrace();
        } catch (IOException e) {
            logger.error(e.toString());
            e.printStackTrace();
        }
        return newFile;
    }

    /**
     * 处理缩略图，并返回新生成图片的相对值路径
     *
     * @param thumbnail 文件流
     * @param targetAddr 相对路径，存储的文件夹地址
     * @return
     */
    public static String generateThumbnail(File thumbnail, String targetAddr) {
        // 获取不重复的随机名
        String realFileName = getRandomFileName();
        // 获取文件的扩展名如png,jpg等
        String extension = getFileExtension(thumbnail);
        // 如果目标路径不存在，则自动创建
        makeDirPath(targetAddr);
        // 获取文件存储的相对路径(带文件名)
        String relativeAddr = targetAddr + realFileName + extension;
        logger.debug("current relativeAddr is :" + relativeAddr);
        // 获取文件要保存到的目标路径
        File dest = new File(PathUtil.getImgBasePath() + relativeAddr);
        logger.debug("current complete addr is :" + PathUtil.getImgBasePath() + relativeAddr);
        logger.debug("basePath is :" + basePath);
        // 调用Thumbnails生成带有水印的图片
        try {
            Thumbnails.of(thumbnail).size(200, 200)
                    .watermark(Positions.BOTTOM_RIGHT, ImageIO.read(new File(basePath + "watermark.jpg")), 0.25f)
                    .outputQuality(0.8f).toFile(dest);
        } catch (IOException e) {
            logger.error(e.toString());
            throw new RuntimeException("创建缩略图失败：" + e.toString());
        }
        // 返回图片相对路径地址
        return relativeAddr;
    }
        /**
     * 递归创建目标路径所涉及到的目录
     *
     * @param targetAddr
     */
    private static void makeDirPath(String targetAddr) {
        String realFileParentPath = PathUtil.getImgBasePath() + targetAddr;
        File dirPath = new File(realFileParentPath);
        if (!dirPath.exists()) {
            dirPath.mkdirs();
        }
    }

    /**
     * 获取输入文件流的扩展名
     *
     * @param file
     * @return
     */
    private static String getFileExtension(File file) {
        String fileName = file.getName();
        return fileName.substring(fileName.lastIndexOf("."));
    }

    /**
     * 生成随机文件名，当前年月日小时分钟秒钟+五位随机数
     *
     * @return
     */
    public static String getRandomFileName() {
        // 获取随机的五位数
        int rannum = r.nextInt(89999) + 10000;
        String nowTimeStr = sDateFormat.format(new Date());
        return nowTimeStr + rannum;
    }

    public static void main(String[] args) throws IOException {
        Thumbnails.of(new File("/Users/baidu/work/image/xiaohuangren.jpg")).size(200, 200)
                .watermark(Positions.BOTTOM_RIGHT, ImageIO.read(new File(basePath + "/watermark.jpg")), 0.25f)
                .outputQuality(0.8f).toFile("/Users/baidu/work/image/xiaohuangrennew.jpg");
    }

    /**
     * storePath是文件的路径还是目录的路径， 如果storePath是文件路径则删除该文件，
     * 如果storePath是目录路径则删除该目录下的所有文件
     *
     * @param storePath
     */
    public static void deleteFileOrPath(String storePath) {
        File fileOrPath = new File(PathUtil.getImgBasePath() + storePath);
        if (fileOrPath.exists()) {
            if (fileOrPath.isDirectory()) {
                File files[] = fileOrPath.listFiles();
                for (int i = 0; i < files.length; i++) {
                    files[i].delete();
                }
            }
            fileOrPath.delete();
        }
    }
}
```

这里的实现并不是最终的，其中`generateThumbnail`的第一个参数类型设置为`File`是为了方便测试。

### PathUtil

```java
public class PathUtil {
    //获取系统的文件分隔符
    private static String seperator = System.getProperty("file.separator");

    public static String getImgBasePath() {
        String os = System.getProperty("os.name");
        String basePath = "";
        if (os.toLowerCase().startsWith("win")) {
            basePath = "D:/projectdev/image";
        } else {
            basePath = "/home/yikang/publicPro";
        }
        basePath = basePath.replace("/", seperator);
        return basePath;
    }

    public static String getShopImagePath(long shopId) {
        String imagePath = "/upload/images/item/shop/" + shopId + "/";
        return imagePath.replace("/", seperator);
    }

    public static String getHeadLineImagePath() {
        String imagePath = "/upload/images/item/headtitle/";
        return imagePath.replace("/", seperator);
    }

    public static String getShopCategoryPath() {
        String imagePath = "/upload/images/item/shopcategory/";
        return imagePath.replace("/", seperator);
    }
}
```

## Dao层：新增店铺、更新店铺

```java
public interface ShopDao {
    /**
     * 新增店铺
     */
    int insertShop(Shop shop);

    /**
     * 更新店铺信息
     */
    int updateShop(Shop shop);
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.yikang.o2o.dao.ShopDao">
  
  <insert id="insertShop" useGeneratedKeys="true" keyColumn="shop_id"
      keyProperty="shopId">
      INSERT INTO
      tb_shop(owner_id, area_id, shop_category_id,
      shop_name, shop_desc, shop_addr,
      phone, shop_img, priority,
      create_time, last_edit_time, enable_status,
      advice)
      VALUES
      (#{owner.userId},#{area.areaId},#{shopCategory.shopCategoryId},#{shopName},
      #{shopDesc},#{shopAddr},#{phone},#{shopImg},#{priority},
      #{createTime},#{lastEditTime}, #{enableStatus},#{advice})
   </insert>
   <update id="updateShop" parameterType="com.yikang.o2o.entity.Shop">
      update tb_shop
      <set>
         <if test="shopName != null">shop_name=#{shopName},</if>
         <if test="shopDesc != null">shop_desc=#{shopDesc},</if>
         <if test="shopAddr != null">shop_addr=#{shopAddr},</if>
         <if test="phone != null">phone=#{phone},</if>
         <if test="shopImg != null">shop_img=#{shopImg},</if>
         <if test="priority != null">priority=#{priority},</if>
         <if test="lastEditTime != null">last_edit_time=#{lastEditTime},</if>
         <if test="enableStatus != null">enable_status=#{enableStatus},</if>
         <if test="advice != null">advice=#{advice},</if>
         <if test="area != null">area_id=#{area.areaId},</if>
         <if test="shopCategory != null">shop_category_id=#{shopCategory.shopCategoryId}</if>
      </set>
      where shop_id=#{shopId}
   </update>
</mapper>
```

` useGeneratedKeys="true" keyColumn="shop_id"   keyProperty="shopId"`使用自增主键，当insert语句执行成功时，自增的主键会自动的注入到传入的shop参数，即shopId（传入前shop.shopId为null，执行后shop.shopId为自增的主键）。

## Dto层：ShopExecution

DTO层主要提供的作用：

数据传输对象层，该层负责屏蔽后端的实体层，将UI层需要的数据进行重新的定义和封装，在实际的业务场景下，后端实现或存储的数据远比用户需要的数据要庞大和复杂，所以前端需要的数据相对来说要么是组合的，要么是抽取的，不是完整的，因为我们在设计数据存储格式上都会有一些额外的设计和考虑。

前端的UI层，只是知道DTO的存在，同时前端需要的数据都在一个DTO中，这样，每次调用服务层的时候，只需要调用一次就可以完成所有的业务逻辑操作，而不是原来的直接调用业务逻辑层那样的，需要调用多次，对于分布式场景下，减少服务调用的次数，尤其重要。

### ShopStateEnum

Dto需要包含状态和状态信息，可以将这状态和状态信息封装为枚举类型。

将构造器定义为private，不开放setter，防止第三方修改枚举。

```java
package com.yikang.o2o.enums;

public enum ShopStateEnum {
    CHECK(0, "审核中"), OFFLINE(-1, "非法店铺"), SUCCESS(1, "操作成功"),
    PASS(2, "通过认证"), INNER_ERROR(-1001, "内部系统错误"),
    NULL_SHOPID(-1002, "shopId为空");
    private int state;
    private String stateInfo;

    //设置为private表示不想让第三方定义ShopStateEnum
    private ShopStateEnum(int state, String stateInfo) {
        this.state = state;
        this.stateInfo = stateInfo;
    }

    public static ShopStateEnum stateOf(int state) {
        for (ShopStateEnum stateEnum : values()) {
            if (stateEnum.getState() == state) return stateEnum;
        }
        return null;
    }

    public int getState() {
        return state;
    }

    public String getStateInfo() {
        return stateInfo;
    }
}
```

#### ShopExecution

增删改查可能涉及到多个shop，比如增删改涉及一个shop（大部分情况，小部分情况是多个），查询设计多个shop，因此需要分别引入`shop`和`shopList`。同时成员变量`count`来存储店铺数量。

```java
public class ShopExecution {
    //结果状态
    private int state;
    //状态标识
    private String stateInfo;
    //店铺数量
    private int count;
    //操作的shop（增删改店铺的时候用到）
    private Shop shop;
    //shop列表（查询店铺列表的时候使用）
    private List<Shop> shopList;

    public ShopExecution() {
    }

    //店铺操作失败的时候的构造器
    public ShopExecution(ShopStateEnum stateEnum) {
        this.state = stateEnum.getState();
        this.stateInfo = stateEnum.getStateInfo();
    }

    //店铺操作成功的构造器
    public ShopExecution(ShopStateEnum stateEnum, Shop shop) {
        this.state = stateEnum.getState();
        this.stateInfo = stateEnum.getStateInfo();
        this.shop = shop;
    }

    //店铺操作成功的构造器
    public ShopExecution(ShopStateEnum stateEnum, List<Shop> shopList) {
        this.state = stateEnum.getState();
        this.stateInfo = stateEnum.getStateInfo();
        this.shopList = shopList;
    }
    //getter and setter
}
```

## Service层：店铺注册

```java
public interface ShopService {
    ShopExecution addShop(Shop shop, File shopImg);
}
```
注意，因为addShop分为多个步骤，需要保证原子性，因此需要添加`@Transactional`注解。

当抛出RuntimeException异常或其子类时，事务会回滚。注意，抛出Exception并不会使事务回滚。

```java
@Service
public class ShopServiceImpl implements ShopService {


    @Autowired
    private ShopDao shopDao;

    @Override
    @Transactional
    public ShopExecution addShop(Shop shop, File shopImg) {
        //空值判断
        if (shop == null) {
            return new ShopExecution(ShopStateEnum.NULL_SHOP);
        }
        try {
            //初始值
            shop.setEnableStatus(0);
            shop.setCreateTime(new Date());
            shop.setLastEditTime(new Date());
            //添加店铺信息
            int effectedNum = shopDao.insertShop(shop);
            if (effectedNum <= 0) {
                //默认情况下SpringAOP只捕获ShopOperationException的异常
                //抛出Exception不会回滚
                throw new ShopOperationException("店铺创建失败");
            } else {
                if (shopImg != null) {
                    //存储图片
                    try {
                        addShopImg(shop, shopImg);
                    } catch (Exception e) {
                        throw new ShopOperationException("addShopImg error: " + e.getMessage());
                    }
                }
                //跟新店铺的图片地址
                effectedNum = shopDao.updateShop(shop);
                if (effectedNum <= 0) {
                    throw new ShopOperationException("跟新图片地址失败");
                }
            }
        } catch (Exception e) {
            //抛出异常，交给controller处理
            throw new ShopOperationException("addShop error: " + e.getMessage());
        }
        return new ShopExecution(ShopStateEnum.CHECK, shop);
    }

    private void addShopImg(Shop shop, File shopImg) {
        //获取shop的图片的目录的相对值路径
        String dest = PathUtil.getShopImagePath(shop.getShopId());
        String shopImgAddr = ImageUtil.generateThumbnail(shopImg, dest);
        shop.setShopImg(shopImgAddr);
    }
}
```

### 测试

注意，在test目录下执行junit测试，`Thread.currentThread().getContextClassLoader().getResource("").getPath()`输出的是`/home/yikang/IdeaProjects/o2o/target/test-classes/`路径，因此如果仅仅将资源文件放在src/main/resources是无法访问到的，应该放在src/test/resources目录下，编译整个项目后，在/target/test-classes目录下才会生成相应的资源文件。

```java
@Test
public void testAddShop() throws ShopOperationException, FileNotFoundException {
    Shop shop = new Shop();
    PersonInfo owner = new PersonInfo();
    Area area = new Area();
    ShopCategory shopCategory = new ShopCategory();
    owner.setUserId(1L);
    area.setAreaId(2);
    shopCategory.setShopCategoryId(1L);
    shop.setOwner(owner);
    shop.setArea(area);
    shop.setShopCategory(shopCategory);
    shop.setShopName("测试的店铺3");
    shop.setShopDesc("test3");
    shop.setShopAddr("test3");
    shop.setPhone("test3");
    shop.setCreateTime(new Date());
    shop.setEnableStatus(ShopStateEnum.CHECK.getState());
    shop.setAdvice("审核中");
    File shopImg = new File("/home/yikang/Picture/user_800_450.jpg");
    ShopExecution se = shopService.addShop(shop, shopImg);
    assertEquals(ShopStateEnum.CHECK.getState(), se.getState());
}
```

## HttpServletRequestUtil

简单的一个工具类，帮助从HttpServletRequest中获取对应的字段。

```java
public class HttpServletRequestUtil {
    public static int getInt(HttpServletRequest request, String key) {
        try {
            return Integer.decode(request.getParameter(key));
        } catch (Exception e) {
            return -1;
        }
    }

    public static long getLong(HttpServletRequest request, String key) {
        try {
            return Long.valueOf(request.getParameter(key));
        } catch (Exception e) {
            return -1;
        }
    }

    public static Double getDouble(HttpServletRequest request, String key) {
        try {
            return Double.valueOf(request.getParameter(key));
        } catch (Exception e) {
            return -1d;
        }
    }

    public static boolean getBoolean(HttpServletRequest request, String key) {
        try {
            return Boolean.valueOf(request.getParameter(key));
        } catch (Exception e) {
            return false;
        }
    }

    public static String getString(HttpServletRequest request, String key) {
        try {
            String result = request.getParameter(key);
            if (result != null) {
                result = result.trim();
            }
            if ("".equals(result)) {
                result = null;
            }
            return result;
        } catch (Exception e) {
            return null;
        }
    }
}
```

## Controller：店铺注册

对ImageUtil改动了些，这里省略。

```java
@Autowired
private ShopService shopService;

@RequestMapping(value = "/registershop", method = RequestMethod.POST)
@ResponseBody
private Map<String, Object> registerShop(HttpServletRequest request) {
    //接受并转化相应的参数，包括店铺信息以及图片信息
    Map<String, Object> modelMap = new HashMap<>();
    String shopStr = HttpServletRequestUtil.getString(request, "shopStr");
    ObjectMapper mapper = new ObjectMapper();
    Shop shop = null;
    try {
        shop = mapper.readValue(shopStr, Shop.class);
    } catch (Exception e) {
        modelMap.put("success", false);
        modelMap.put("errMsg", e.getMessage());
    }
    //文件流
    CommonsMultipartFile shopImg = null;
    CommonsMultipartResolver commonsMultipartResolver =
            new CommonsMultipartResolver(request.getSession().getServletContext());
    if (commonsMultipartResolver.isMultipart(request)) {
        MultipartHttpServletRequest multipartHttpServletRequest = (MultipartHttpServletRequest) request;
        shopImg = (CommonsMultipartFile) multipartHttpServletRequest.getFile("shopImg");
    } else {
        modelMap.put("success", false);
        modelMap.put("errMsg", "上传图片不能为空");
    }
    //注册店铺
    if (shop != null && shopImg != null) {
        PersonInfo owner = new PersonInfo();
        owner.setUserId(1L);
        shop.setOwner(owner);
        ShopExecution se = null;
        try {
            se = shopService.addShop(shop, shopImg.getInputStream(), shopImg.getOriginalFilename());
        } catch (IOException e) {
            modelMap.put("success", false);
            modelMap.put("errMsg", e.getMessage());
        }
        if (se.getState() == ShopStateEnum.CHECK.getState()) {
            modelMap.put("success", true);
        } else {
            modelMap.put("success", false);
            modelMap.put("errMsg", se.getStateInfo());
        }
    } else {
        modelMap.put("success", false);
        modelMap.put("errMsg", "请输入店铺信息");
    }
    //返回结果
    return modelMap;
}
```

## 验证码

将下列依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>com.github.penggle</groupId>
    <artifactId>kaptcha</artifactId>
    <version>2.3.2</version>
</dependency>
```

修改web.xml，添加一个用于验证码的servlet：

```xml
<servlet>
    <servlet-name>Kaptcha</servlet-name>
    <servlet-class>com.google.code.kaptcha.servlet.KaptchaServlet</servlet-class>
    <!-- 是否有边框 -->
    <init-param>
        <param-name>kaptcha.border</param-name>
        <param-value>no</param-value>
    </init-param>
    <!-- 字体颜色 -->
    <init-param>
        <param-name>kaptcha.textproducer.font.color</param-name>
        <param-value>red</param-value>
    </init-param>
    <!-- 图片宽度 -->
    <init-param>
        <param-name>kaptcha.image.width</param-name>
        <param-value>135</param-value>
    </init-param>
    <!-- 使用哪些字符生成验证码 -->
    <init-param>
        <param-name>kaptcha.textproducer.char.string</param-name>
        <param-value>ACDEFHKPRSTWX345679</param-value>
    </init-param>
    <!-- 图片高度 -->
    <init-param>
        <param-name>kaptcha.image.height</param-name>
        <param-value>50</param-value>
    </init-param>
    <!-- 字体大小 -->
    <init-param>
        <param-name>kaptcha.textproducer.font.size</param-name>
        <param-value>43</param-value>
    </init-param>
    <!-- 干扰线的颜色 -->
    <init-param>
        <param-name>kaptcha.noise.color</param-name>
        <param-value>black</param-value>
    </init-param>
    <!-- 字符个数 -->
    <init-param>
        <param-name>kaptcha.textproducer.char.length</param-name>
        <param-value>4</param-value>
    </init-param>
    <!-- 字体 -->
    <init-param>
        <param-name>kaptcha.textproducer.font.names</param-name>
        <param-value>Arial</param-value>
    </init-param>
</servlet>
<servlet-mapping>
        <servlet-name>Kaptcha</servlet-name>
        <url-pattern>/Kaptcha</url-pattern>
</servlet-mapping>
```

这样，访问http://localhost:8080/Kaptcha?1就能得到验证码。

注意，前后端没有分离的情况下，img的属性src应该取访问web.xml的路径，比如这种情况：

![](/home/yikang/Picture/2021-03-17 17-43-09屏幕截图.png)

`shopoperation.html`中的验证码的路径应该写为`../../Kapcha?id`。

定义CodeUtil工具类从HttpServletRequest获取验证码的信息，并与HttpServletRequest中的verifyCodeActual字段进行比较。

```java
public class CodeUtil {
   /**
    * 检查验证码是否和预期相符
    * 
    * @param request
    * @return
    */
   public static boolean checkVerifyCode(HttpServletRequest request) {
      String verifyCodeExpected = (String) request.getSession()
            .getAttribute(com.google.code.kaptcha.Constants.KAPTCHA_SESSION_KEY);
      String verifyCodeActual = HttpServletRequestUtil.getString(request, "verifyCodeActual");
      if (verifyCodeActual == null || !verifyCodeActual.equals(verifyCodeExpected)) {
         return false;
      }
      return true;
   }
}
```

## 上传图片的文件流的配置

注意，需要在spring-web.xml中配置文件上传解析器，否则request中的所有key都会为空。

```xml
<!-- 文件上传解析器 -->
<bean id="multipartResolver"
      class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="defaultEncoding" value="utf-8"></property>
    <!-- 1024 * 1024 * 20 = 20M -->
    <property name="maxUploadSize" value="20971520"></property>
    <property name="maxInMemorySize" value="20971520"></property>
</bean>
```

同时需要引入库：

```xml
<!--文件上传-->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.2</version>
</dependency>
```

至于获取图片文件流`CommonsMultipartFile`，需要用以下方式：

```java
//获取request中的名为shopImg的文件流
CommonsMultipartFile shopImg = null;
CommonsMultipartResolver commonsMultipartResolver =
        new CommonsMultipartResolver(request.getSession().getServletContext());
if (commonsMultipartResolver.isMultipart(request)) {
    MultipartHttpServletRequest multipartHttpServletRequest = (MultipartHttpServletRequest) request;
    shopImg = (CommonsMultipartFile) multipartHttpServletRequest.getFile("shopImg");
} else {
    modelMap.put("success", false);
    modelMap.put("errMsg", "上传图片不能为空");
}
```