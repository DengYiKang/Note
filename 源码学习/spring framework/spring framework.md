# spring framework

## 编译源码

进入spring源码的目录，修改build.gradle文件，修改源为aliyun镜像。

```
buildscript {
	repositories{
		maven { url 'http://maven.aliyun.com/nexus/content/groups/public/'}
    	maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter'}
	}
	dependencies {
		classpath 'org.asciidoctor:asciidoctorj-pdf:1.5.0-alpha.16'
		classpath 'io.spring.asciidoctor:spring-asciidoctor-extensions:0.1.3.RELEASE'
	}
}
//......
configure(allprojects) { project ->
//......
		repositories {
			maven { url 'http://maven.aliyun.com/nexus/content/groups/public/'}
    		maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter'}
			mavenCentral()
			maven { url "https://repo.spring.io/libs-spring-framework-build" }
		}
}
```

## Bean与BeanDefinition

### Bean是Spring的一等公民

Bean本质上就是java对象，只是这个对象的声明周期由容器来管理。

不需要为了创建Bean而在原来的java类上添加任何额外的限制。（低侵入性）

对java对象的控制方式体现在配置上。

### BeanDefinition

根据配置，生成用来描述Bean的BeanDefinition，常用属性：

+ 作用范围scope（`@Scope`）
  + singleton：单例
  + prototype：每次获取Bean的时候会有一个新的实例
  + request：request表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP request内有效
  + session：session作用域表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP session内有效
  + globalsession：global session作用域类似于标准的HTTP Session作用域，不过它仅仅在基于portlet的web应用中才有意义
+ 懒加载lazy-init（`@Lazy`）：决定Bean实例是否延迟加载
+ 首选primary（`@Primary`）：设置为true的bean会是优先的实现类（在实现类有多个的情况下）
+ factory-bean和factory-method（`@Configuration`和`@Bean`）

这里使用XML配置文件来举例子：

```xml
<bean id="welcomeService" class="com.yikang.service.impl.WelcomeServiceImpl"/>
<!-- 1.使用类的无参构造函数创建 -->
<bean id="user1" class="com.yikang.entity.User" scope="singleton" lazy-init="true" primary="true"/>
<alias name="user1" alias="userAlias1,userAlias2,userAlias3"/>
<!-- 2.使用静态工厂进行创建 -->
<!-- class的值不是写User对象的全路径，而是写静态工厂的全路径 -->
<!-- factory-method的值写要调用的方法 -->
<bean id="user2" class="com.yikang.entity.factory.StaticFactory" factory-method="getUser" scope="singleton"/>
<!-- 3.使用实例工厂进行创建 -->
<!-- 需要先创建factoryBean对象，再通过factoryBean对象进行调用 -->
<bean id="userFactory" class="com.yikang.entity.factory.UserFactory"/>
<bean id="user3" factory-bean="userFactory" factory-method="getUser" scope="prototype"/>
<bean id="userFactoryBean" class="com.yikang.entity.factory.UserFactoryBean"/>
```

user1为`User`实例，单例，懒加载，优先的实现类。

user2是通过静态工厂`StaticFactory`的静态方法`getUser`来创建的，他的类型也是`User`，单例。

user3是通过工厂`UserFactory`的方法`getUser`创建的，注意`UserFactory#getUser`并不是静态的，因此需要先将`UserFactory`实例化，然后在`factory-bean`属性中指定工厂bean。

> 注意，单例是对bean而言的，并非对类类型。

对于userFactoryBean，它是UserFactoryBean类型，实现了`FActoryBean<T>`接口：

```java
public class UserFactoryBean implements FactoryBean<User> {
   @Override
   public User getObject() throws Exception {
      return new User();
   }

   @Override
   public Class<?> getObjectType() {
      return User.class;
   }
}
```

调用`applicationContext.getBean("userFactoryBean")`，返回的是`User`类型的对象；

调用`applicationContext.getBean("&userFactoryBean")`，返回的是`UserFactoryBean`类型的对象。

但是要注意，返回的不管是`User`还是`UserFactoryBean`，都是单例的（如果设置为prototype，那么两者都是prototype的）。

> Spring中存在两个名字很相似的接口：BeanFactory和FactoryBean。
>
> BeanFactory是用于实现Spring IOC容器的接口，而FactoryBean只是一个作为简单工厂的Bean接口。

## ApplicationContext常用容器

### 传统的基于XML配置的经典容器

+ FileSystemXmlApplicationContext：从文件系统加载配置
+ ClassPathXmlApplicationContex：从classpath加载配置
+ XmlWebApplicationContext：用于Web应用程序的容器

### 目前比较流行的容器

+ AnnotationConfigServletWebServerApplicationContext
+ AnnotationConfigReactiveWebServerApplicationContext
+ AnnotationConfigApplicationContext

### 容器的共性

refresh()大致功能：

+ 容器初始化、配置解析
+ BeanFactoryPostProcessor和BeanPostProcessor的注册和激活
+ 国际化配置
+ 等等

## 根据资源地址自动选择正确的Resource

### Ant

路径匹配表达式，用来对URI进行匹配：

+ ?匹配任何单字符
+ *匹配0或者任意数量的字符
+ **匹配0或者更多的目录

| 通配符 | 示例          | 说明                                                     |
| ------ | ------------- | -------------------------------------------------------- |
| ?      | /test/p?ocess | 匹配/test/process，但是不匹配/test/pocess                |
| *      | /test/*.jsp   | 匹配项目路径下所有在test路径下的.jsp文件                 |
| *      | /test/*/path  | /test/path、/test/a/path都匹配，但是不匹配/test/a/b/path |
| **     | /test/**/path | /test/path、/test/a/path、/test/a/b/path都匹配           |

### 强大的加载资源方式

+ 自动识别"classpath:"、"file:"等资源地址前缀
+ 支持自动解析Ant风格带通配符的资源地址

### ResourceLoader

实现不同的ResourceLoader加载策略，按需返回特定类型的Resource

### BeanDefinitionReader

+ 读取BeanDefinition
+ BeanDefinitionRegistry

关键词：

+ location
+ Resource
+ Resourceloader
+ BeanDefinitionReader
+ BeanDefinitionRegistry
+ DefaultListableBeanFactory

## 容器初始化主要做的事情

首先将配置文件读取到内存中，这些配置将会被当做Resource对象，然后被解析成BeanDefinition实例，最后注册到spring容器中。

## 后置处理器PostProcessor

本身也是一种需要注册到容器里的Bean

+ 其里面的方法会在特定的时机被容器调用
+ 实现不改变容器或者Bean核心逻辑的情况下对Bean进行扩展
+ 对Bean进行包装，影响其行为、修改Bean的内容等

### 种类

大类分为容器级别的后置处理器以及Bean级别的后置处理器

+ BeanDefinitionRegistryPostProcessor
+ BeanFactoryPostProcessor
+ BeanPostProcessor

## Spring的时间驱动模型

事件驱动模型的三大组成部分

+ 事件：ApplicationEvent抽象类
+ 事件监听器：ApplicationLIstener
+ 事件发布器：Publish以及Multicaster

## Spring是否支持所有循环依赖的情况

循环依赖的情况如下：

+ 构造器循环依赖（singleton、prototype）
+ Setter注入循环依赖（singleton、prototype）

### Spring不支持prototype的循环依赖

因为没有设置三级缓存进行支持

+ 只能通过将Bean的名字放入缓存里阻断无限循环

### Spring不支持单例构造器的循环依赖

### Spring支持单例的setter注入循环依赖

## keyword

+ Aware
+ AbstractApplicationContext#refresh：（refresh只加载非延时singleton的bean，对于prototype等类型的bean不会加载，只能通过显式的方式加载，如getBean方法）
+ populateBean
+ SpringAOP的实现原理之CGLIB动态代理
  + 不要求被代理类实现接口
  + 内部主要封装了ASM Java字节码操控框架
  + 动态生成子类以覆盖非final的方法，绑定钩子回调自定义拦截器

## 术语补充

+ 组件扫描：自动发现应用容器中需要创建的Bean（即扫描所有的class查看是否被`@Controller`等修饰，然后初始化）
+ 自动装配：自动满足Bean之间的依赖（即依赖注入，`@Autowired`等）