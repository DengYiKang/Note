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