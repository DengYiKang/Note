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

## 容器初始化主要做的事情

<img src="../../pic/203.jpg" style="zoom:80%;" />

## BeanDefinition

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

对于userFactoryBean，它是UserFactoryBean类型，实现了`FactoryBean<T>`接口：

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
> BeanFactory是用于实现Spring IOC容器的接口，而FactoryBean只是一个作为简单工厂的Bean接口，只有getObject、getObjectType、isSington方法。

### more about BeanDefinition

<img src="../../pic/204.png" style="zoom:80%;" />

+ RootBeanDefinition可以单独作为一个BeanDefinition，也可以作为其他BeanDefinition的父类。但是他不能作为其他BeanDefinition的子类（在setParentName的时候，会抛出一个异常）
+ ChildBeanDefinition相当于一个子类，不可以单独存在，必须要依赖一个父BeanDetintion。（parentName属性是通过构造方法设置的，而且并没有提供无参构造方法)
+ GenericBeanDefinition是一个通用类，观察他源码跟ChildBeanDefinition很像，可以替代ChildBeanDefinition。
+ RootBeanDefinition的域和方法远多于GenericBeanDefinition，定义root bean时应该尽量使用RootBeanDefinition。
+ BeanDefinition之间的关系不是通过继承来表现的，而是通过parentName属性表现的。

## IOC容器

### BeanFactory

<img src="../../pic/205.png" style="zoom:80%;" />

需要注意的是`FACTORY_BEAN_PREFIX`这个域，这个域提供获取FactoryBean的方法，如果使用bean的名字检索FactoryBean得到的对象是工厂生成的对象，如果需要得到工厂本身，那么需要转义，即bean名字前加个前缀"&"。

bean都是由BeanFactory的实现类来管理的。

### 简单容器



<img src="../../pic/206.png" style="zoom:80%;" />

#### ListableBeanFactory

<img src="../../pic/208.png" style="zoom:80%;" />

这个接口最大的特点就是可以根据需要列出对应的BeanDefinition、BeanName列表。

#### HierarchicalBeanFactory

<img src="../../pic/209.png" style="zoom:80%;" />

这个接口使得BeanFactory之间具备层级关系。

注意`containsLocalBean`只会在本层的容器中查找，不会去父类容器查找。

#### AutowireCapableBeanFactory

<img src="../../pic/210.png" style="zoom:80%;" />

从宏观上看，AutowireCapableBeanFactory提供了如下能力：

 1 为已经实例化的对象装配属性，这些属性对象都是Spring管理的；

 2 实例化一个Bean，并自动装配，这些被装配的属性对象都是Spring管理的，但是实例化的Bean可以不被Spring管理（这点特别重要）。所以这个接口提供功能就是自动装配bean相关的

 (自动装配的原对象可以不在Spring的IOC容器里，但是需要被依赖注入的成员，就必须是Spring容器管辖的Bean)
 此接口主要是针对框架之外，没有向Spring托管Bean的应用。通过暴露此功能，Spring框架之外的程序，也能具有自动装配的能力（此接口赋予它的）。

 可以使用这个接口集成其它框架。捆绑并填充（注入）并不由Spring管理生命周期并已存在的实例.像集成WebWork的Actions和Tapestry Page就很实用。
 一般应用开发者不会使用这个接口,所以像ApplicationContext这样的外观实现类不会实现这个接口。

### 高级容器

<img src="../../pic/207.png"  />

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
      MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
          ...;
      }
```

#### 传统的基于XML配置的经典容器

+ FileSystemXmlApplicationContext：从文件系统加载配置
+ ClassPathXmlApplicationContex：从classpath加载配置
+ XmlWebApplicationContext：用于Web应用程序的容器

#### 目前比较流行的容器

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

### Resource

Resource代表底层外部资源，提供了对底层外部资源的一致性访问接口。

<img src="../../pic/211.png" style="zoom:80%;" />

### ResourceLoader

实现不同的ResourceLoader加载策略，按需返回特定类型的Resource，可以看做是一个生产Resource的工厂类，根据传入一个资源路径返回对应类型的Resource。

<img src="../../pic/212.png" style="zoom:80%;" />

其中，DefaultResourceLoader的getResource方法只支持单次解析，如果需要解析多条location，那么需要重复调用多次。

如果需要一次性解析多个location，采用Ant风格的location，建议使用实现了ResourcePatternResolver接口的PathMatchingResourcePatternResolver。

DefaultResourceLoader的getResource方法：

```java
//获取Resource的具体实现类实例
@Override
public Resource getResource(String location) {
   Assert.notNull(location, "Location must not be null");
   //ProtocolResolver ，用户自定义协议资源解决策略
   for (ProtocolResolver protocolResolver : getProtocolResolvers()) {
      Resource resource = protocolResolver.resolve(location, this);
      if (resource != null) {
         return resource;
      }
   }
   //如果是以/开头，则构造ClassPathContextResource返回
   if (location.startsWith("/")) {
      return getResourceByPath(location);
   }
   //若以classpath:开头，则构造ClassPathResource 类型资源并返回，在构造该资源时，
   //通过 getClassLoader()获取当前的ClassLoader
   else if (location.startsWith(CLASSPATH_URL_PREFIX)) {
      return new ClassPathResource(location.substring(CLASSPATH_URL_PREFIX.length()), getClassLoader());
   }
   else {
      // 构造 URL ，尝试通过它进行资源定位，若没有抛出 MalformedURLException 异常，
      // 则判断是否为 FileURL , 如果是则构造 FileUrlResource 类型资源，否则构造 UrlResource。
      // 若在加载过程中抛出 MalformedURLException 异常，
      // 则委派 getResourceByPath() 实现资源定位加载
      try {
         // Try to parse the location as a URL...
         URL url = new URL(location);
         return (ResourceUtils.isFileURL(url) ? new FileUrlResource(url) : new UrlResource(url));
      }
      catch (MalformedURLException ex) {
         // No URL -> resolve as resource path.
         return getResourceByPath(location);
      }
   }
} 
```
 因为PathMatchingResourcePatternResolver的getResources方法比较复杂，这里只贴ResourcePatternResolver接口，可以看出getResources方法的返回值是Resource数组，接受的location可以是Ant风格的字符串。

```java
public interface ResourcePatternResolver extends ResourceLoader {

   /**
    * Pseudo URL prefix for all matching resources from the class path: "classpath*:"
    * This differs from ResourceLoader's classpath URL prefix in that it
    * retrieves all matching resources for a given name (e.g. "/beans.xml"),
    * for example in the root of all deployed JAR files.
    * @see org.springframework.core.io.ResourceLoader#CLASSPATH_URL_PREFIX
    */
   String CLASSPATH_ALL_URL_PREFIX = "classpath*:";

   /**
    * Resolve the given location pattern into Resource objects.
    * <p>Overlapping resource entries that point to the same physical
    * resource should be avoided, as far as possible. The result should
    * have set semantics.
    * @param locationPattern the location pattern to resolve
    * @return the corresponding Resource objects
    * @throws IOException in case of I/O errors
    */
   Resource[] getResources(String locationPattern) throws IOException;

}
```

## BeanDefinitionReader

BeanDefinitionReader是一个接口，用于从单个或多个配置文件（Resource）中加载BeanDefinition（即读取配置文件，生成对应的BeanDefinition）。

<img src="../../pic/214.png" style="zoom:80%;" />

以下是BeanDefinitionReader的继承关系图：

<img src="../../pic/215.png" style="zoom:80%;" />

关键词：

+ location
+ Resource
+ Resourceloader
+ BeanDefinitionReader
+ BeanDefinitionRegistry
+ DefaultListableBeanFactory

## BeanDefinitionRegistry

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