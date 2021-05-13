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

首先将配置文件读取到内存中，这些配置将会被当做Resource对象，然后被解析成BeanDefinition实例，最后注册到spring容器中。

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

其中，XmlBeanDefinitionReader用于读取xml配置文件，PropertiesBeanDefinitionReader用于读取properties配置文件，AnnotatedBeanDefinition用于读取注解相关的配置。

关键词：

+ location
+ Resource
+ Resourceloader
+ BeanDefinitionReader
+ BeanDefinitionRegistry
+ DefaultListableBeanFactory

## BeanDefinitionRegistry

<img src="../../pic/267.png" style="zoom:80%;" />

可以查看它的一个实现类DefaultListableBeanFactory，简答来说，最终将注册的bean放入一个ConcurrentHashMap中存储：

```java
/** Map of bean definition objects, keyed by bean name. */
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
```

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

## Aware

Aware是一个空接口，用于标记。

<img src="../../pic/268.png" style="zoom:80%;" />

实现了Aware接口的类可以获取不同的容器实例并对其进行操作，例如实现了ApplicationContextAware接口的类可以获得ApplicationContext容器，实现了BeanFactoryAware接口的类可以获得BeanFactory容器。

每个Aware接口都有setter方法，例如ApplicationContextAware接口：

<img src="../../pic/269.png" style="zoom:80%;" />

spring容器会自动调用这些setter方法。

## Spring的事件驱动模型

事件驱动模型的三大组成部分

+ 事件：ApplicationEvent抽象类
+ 事件监听器：ApplicationLIstener
+ 事件发布器：Publisher以及Multicaster

### 事件

<img src="../../pic/270.png" style="zoom:80%;" />

由名字可以看出他们的作用：

+ ContextStartedEvent：容器启动后触发的事件
+ ContextStoppedEvent：容器停止后触发的事件
+ ContextClosedEvent：容器关闭后触发的事件
+ ContextRefreshedEvent：容器初始化或刷新后触发的事件

查看ApplicationEvent的构造器：

```java
public ApplicationEvent(Object source) {
   super(source);
   this.timestamp = System.currentTimeMillis();
}
```

传入的source为事件源。

### 监听器

<img src="../../pic/271.png" style="zoom:80%;" />

ApplicationListener接口只有一个处理event的方法。

#### SmartApplicationListener

额外实现了Order接口，因此该类监听器具备排序的功能。

<img src="../../pic/272.png" style="zoom:80%;" />

`supportsEventType`和`supportsSourceType`方法支持对事件、源的过滤。

#### GenericApplicationListener

<img src="../../pic/273.png" style="zoom:80%;" />

跟SmartApplicationListener非常类似，只不过supportsEventType的接受参数为ResolvableType类型，它可以获取泛型信息。

### 发布器

#### ApplicationEventPublisher

<img src="../../pic/274.png" style="zoom:80%;" />

#### ApplicationEventMulticaster

<img src="../../pic/275.png" style="zoom:80%;" />

注意，ApplicationContext继承了ApplicationEventPublisher接口。

ApplicationEventPublisher只有发布的功能，而ApplicationEventMulticaster拥有发布、维护监听器等功能。

Spring为什么定义了Multicaster还要定义Publisher呢？设计者只想让ApplicationContext拥有publish的功能，而且Multicaster可以作为Publisher的代理类。

## Spring容器的刷新

以下是刷新的步骤：

+ prepareRefresh：刷新前的工作准备
  + 调用容器准备刷新的方法，获取容器的当前时间，同时给容器设置同步标识（容器的状态设置为激活）
  + 初始化Enviroment的propertySources属性
  + 校验Enviroment的requiredProperties是否都存在
  + 检查是否有监听器，如果有，那么将他们都加入到一个set中
  + 创建事件的集合
+ obtainFreshBeanFactory：获取子类刷新后的内部beanFactory实例
  + 调用子类实现的refreshBeanFactory()方法，Bean定义资源文件的载入从子类的refreshBeanFactory（）方法启动，里面有抽象方法；针对xml配置，最终创建内部容器，该容器负责Bean的创建与管理，此步会进行BeanDefinition的注册
+ prepareBeanFactory：为容器注册必要的系统级别的Bean，例如classloader，beanfactoryPostProcessor等
  + 设置内部bean工厂使用容器的类加载器
  + 设置beanFactory的表达式语言处理器，Spring3开始增加了对语言表达式的支持，默认可以使用#{bean.xxx}的形式来调用相关属性值
  + 为beanFactory增加一个默认的propertyEditor
  + 添加beanPostProcessor：当应用程序定义的Bean实现ApplicationContextAware接口时注入ApplicationContext对象
  + 如果某个bean依赖以下几个接口的实现类，在自动装配的时候忽略它们，Spring会通过其他方式来处理这些依赖：
    + EnviromentAware.class
    + EmbeddedValueResolverAware.class
    + ResourceLoaderAware.class
    + ApplicationEventPublisherAware.class
    + MessageSourceAware.class
    + ApplicationContextAware.class
  + 修正部分依赖
  + 添加beanPostProcessor：用于检测内部bean是否是ApplicationListener，如果是，那么加入到事件监听者队列
  + 注册默认enviroment环境bean
+ postProcessBeanFactory：允许容器的子类去注册postProcessor，是一个钩子方法
+ invokeBeanFactoryPostProcessors：调用容器注册的容器级别的后置处理器
  + 实例化并调用所有已注册的BeanFactoryPostProcessor的bean，如果已给出顺序（Order接口），将按照顺序调用。
    + BeanFacotryPostProcessors主要分为两类，一类是BeanDefinitionRegistry的BeanFactoryPostProcessor，另外一类是常规的BeanFactoryPostProcessor。 优先处理前者。
    + BeanDefinitionRegistry将扫描所有的BeanDefinition并注册到容器之中。
+ registerBeanPostProcessors：注册拦截bean创建过程的BeanPostProcessor（例如AOP织入）
+ initMessageSource：初始化国际化配置
+ initApplicationEventMulticaster：初始化事件发布者组件
+ onRefresh：在单例Bean初始化之前预留给子类初始化其他特殊bean口子，钩子方法
  + 该方法需要在所有单例bean初始化之前调用
  + 比如Web容器就会去初始化一些和主题展示相关的bean（ThemeSource）
+ registerListeners：向前面的事件发布者组件注册事件监听者
+ finishBeanFactoryInitialization：设置系统级别的服务，实例化所有非懒加载的单例
+ finishRefresh：触发初始化完成的回调方法，并发布容器刷新完成的事件给监听器
+ resetCommonCaches：重置spring内核中的共用缓存

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