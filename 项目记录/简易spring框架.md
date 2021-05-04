# 简易spring框架

## 基本雏形

新建maven项目，不用任何模板。

### pom

编写pom文件：

添加了servlet、jsp的依赖，以及maven compile。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.yikang</groupId>
    <artifactId>simpleframework</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>
        <!-- https://mvnrepository.com/artifact/javax.servlet.jsp/javax.servlet.jsp-api -->
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>javax.servlet.jsp-api</artifactId>
            <version>2.3.3</version>
            <scope>provided</scope>
        </dependency>

    </dependencies>

    <build>
        <finalName>simpleframework</finalName>
        <pluginManagement>
            <plugins>
                <plugin>
                    <!-- https://mvnrepository.com/artifact/org.apache.maven.plugins/maven-compiler-plugin -->
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.1</version>
                    <configuration>
                        <source>8</source>
                        <target>8</target>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>

</project>
```

### HelloServlet

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String name = "简易框架";
        req.setAttribute("name", name);
        req.getRequestDispatcher("/WEB-INF/jsp/hello.jsp").forward(req, resp);
    }
}
```

### hello.jsp

```html
<%@ page pageEncoding="UTF-8" %>
<html>
<head>
    <title>Hello</title>
</head>
<body>
<h1>Hello!</h1>
<h2>${name}</h2>
</body>
</html>
```

### tomcat配置

<img src="../pic/157.png" style="zoom:67%;" />

### 启动测试

运行tomcat插件，访问[地址](http://localhost:8080/simpleframework/hello)。

## 业务层基本架构搭建

### 拦截请求

拦截请求，根据不同的url转发给不同的controller处理。

```java
/**
 * 这个类起拦截功能，根据不同的url转发到不同的controller处理
 */
@WebServlet("/")
//注意，不能拦截"/*"，否则会进入死循环。因为"/*"会拦截转发行为（forward hello.jsp）
//"/"不会拦截jsp请求
public class DispatcherServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println(req.getServletPath());
        System.out.println(req.getMethod());
        if (req.getServletPath().equals("/frontend/getmainpageinfo") && req.getMethod().equals("GET")) {
            new MainPageController().getMainPageInfo(req, resp);
        } else if (req.getServletPath().equals("/superadmin/addheadline") && req.getMethod().equals("POST")) {
            new HeadLineOperationController().addHeadLine(req, resp);
        }
    }
}
```

### service层

```java
public interface HeadLineShopCategoryCombineService {
    Result<MainPageInfoDTO> getMainPageInfo();
}
```

```java
public interface HeadLineService {
    Result<Boolean> addHeadLine(HeadLine headLine);

    Result<Boolean> removeHeadLine(int headLineId);

    Result<Boolean> modifyHeadLine(HeadLine headLine);

    Result<HeadLine> queryHeadLineById(int headLineId);

    Result<List<HeadLine>> queryHeadLineList(HeadLine headLineCondition, int pageIndex, int pageSize);
}
```

```java
public interface ShopCategoryService {
    Result<Boolean> addShopCategory(ShopCategory shopCategory);

    Result<Boolean> removeShopCategory(int shopCategoryId);

    Result<Boolean> modifyShopCategory(ShopCategory shopCategory);

    Result<ShopCategory> queryShopCategoryById(int shopCategoryId);

    Result<List<ShopCategory>> queryShopCategory(ShopCategory shopCategoryCondition, int pageIndex, int pageSize);
}
```

### controller层

主要有处理主页、处理headline（包含增删查改）、处理shopcategory（包含增删查改），各个方法是各service加上参数检验的代理。

## 根据package获取类集合

根据包名获取该包下的所有类。

首先需要获取类的加载器，以获取项目发布的实际路径，为什么不让用户传入绝对路径？因为不同机器之间的路径可能不同，且如果打的是war包或者jar包，那么根本找不到路径，因此通用的做法是通过项目的类加载器来获取。

根据包名，类加载器可以返回对应的URL，然后从这个URL中递归遍历查找class文件，将他们都加入集合中。

```java
package org.simpleframework.util;

import lombok.extern.slf4j.Slf4j;

import java.io.File;
import java.io.FileFilter;
import java.net.URL;
import java.util.HashSet;
import java.util.Set;

@Slf4j
public class ClassUtil {

    public static final String FILE_PROTOCOL = "file";

    /**
     * 获取包下类集合
     * 1、获取到类的加载器，以获取项目发布的实际路径
     * 为什么不让用户传入绝对路径？因为不同机器之间的路径可能不同；如果打的是war包或者jar包，那么根本找不到路径
     * 因此通用的做法是通过项目的类加载器来获取
     * 2、通过类加载器获取到加载的资源信息
     * 3、依据不同的资源类型，采用不同的方式获取资源的集合
     *
     * @param packageName 包名
     * @return 类集合
     */
    public static Set<Class<?>> extractPackageClass(String packageName) {
        ClassLoader classLoader = getClassLoader();
        //注意包名中的点替换成反斜杠
        URL url = classLoader.getResource(packageName.replace(".", File.separator));
        if (url == null) {
            log.warn("unable to retrieve anything from package:" + packageName);
            return null;
        }
        Set<Class<?>> classSet = null;
        //过滤出本地文件类型的资源
        if (url.getProtocol().equalsIgnoreCase(FILE_PROTOCOL)) {
            classSet = new HashSet<>();
            File packageDirectory = new File(url.getPath());
            extractClassFile(classSet, packageDirectory, packageName);
        }
        //TODO 此处可以加入针对其他类型资源的处理
        return classSet;
    }

    /**
     * 递归获取目标package里面的所有class文件（包括子package里的class文件）
     *
     * @param classSet    装载目标的集合
     * @param fileSource  文件或者目录
     * @param packageName 报名
     */
    private static void extractClassFile(Set<Class<?>> classSet, File fileSource, String packageName) {
        if (!fileSource.isDirectory()) {
            return;
        }
        //如果是文件夹，则调用其listFiles方法获取文件夹下的文件或文件夹
        File[] files = fileSource.listFiles(new FileFilter() {
            @Override
            public boolean accept(File file) {
                if (file.isDirectory()) {
                    return true;
                } else {
                    //获取文件的绝对值路径
                    String absolutePath = file.getAbsolutePath();
                    if (absolutePath.endsWith(".class")) {
                        //若是class文件，则直接加载
                        addToClassSet(absolutePath);
                    }
                }
                return false;
            }

            //根据class文件的绝对值路径，获取并生成class对象，并放入classSet中
            private void addToClassSet(String absolutePath) {
                //1.从class文件的绝对值路径里提取出包含了package的类名
                //如/home/yikang/IdeaProjects/springframework/simpleframework/target/classes/com/yikang/entity/dto/MainPageInfoDTO.class
                //需要弄成com.yikang.entity.dto.MainPageInfoDTO
                absolutePath = absolutePath.replace(File.separator, ".");
                String className = absolutePath.substring(absolutePath.indexOf(packageName));
                className = className.substring(0, className.lastIndexOf("."));
                //2、通过反射机制获取对应的class对象并加入到classSet里
                Class targetClass = loadClass(className);
                classSet.add(targetClass);
            }
        });
        if (files != null) {
            for (File f : files) {
                extractClassFile(classSet, f, packageName);
            }
        }
    }

    /**
     * 获取classLoader
     *
     * @return classLoader
     */
    public static ClassLoader getClassLoader() {
        return Thread.currentThread().getContextClassLoader();
    }

    /**
     * 根据类的全名获取class对象
     *
     * @param className 类的全名
     * @return Class
     */
    public static Class<?> loadClass(String className) {
        try {
            return Class.forName(className);
        } catch (ClassNotFoundException e) {
            log.error("load class error", e);
            throw new RuntimeException(e);
        }
    }
}
```

## 无视反射和序列化攻击的单例

一般有两种方式实现单例，一种使用静态初始化，一种使用锁+二重检查：

```java
public class StarvingSingleton {
    private static final StarvingSingleton starvingSingleton = new StarvingSingleton();
    private StarvingSingleton(){ }
    public static StarvingSingleton getInstance(){
        return starvingSingleton;
    }
}
```

```java
public class LazyDoubleCheckSingleton {
    private volatile static LazyDoubleCheckSingleton instance;

    private LazyDoubleCheckSingleton(){}

    public static LazyDoubleCheckSingleton getInstance(){

        //第一次检测
        if (instance==null){
            //同步
            synchronized (LazyDoubleCheckSingleton.class){
                if (instance == null){
                    //memory = allocate(); //1.分配对象内存空间
                    //instance(memory);    //2.初始化对象
                    //instance = memory;   //3.设置instance指向刚分配的内存地址，此时instance！=null
                    instance = new LazyDoubleCheckSingleton();
                }
            }
        }
        return instance;
    }
}
```

> 注意，在第二种方式里，instance必须为volatile的，因为可能会发生指令重排，导致可能会发生instance有值而并未初始化完成的情况。

这些都能被反射攻破。

对于第一种，原理在于将构造函数设置为private，静态初始化，那么反射直接调用该静态的构造函数就能攻破了。

对于第二种也是一样的情况。

对于序列化攻击，序列化会通过反射调用无参数的构造方法创建一个新的对象。

那么如何实现出无视反射和序列化攻击的单例呢？使用枚举，枚举不允许反射来创建对象。

```java
public class EnumStarvingSingleton {
    private EnumStarvingSingleton() {
    }

    public static EnumStarvingSingleton getInstance() {
        return ContainerHolder.HOLDER.instance;
    }

    private enum ContainerHolder {
        HOLDER;
        private EnumStarvingSingleton instance;

        ContainerHolder() {
            instance = new EnumStarvingSingleton();
        }
    }

}
```

如果使用反射获取到构造函数，那么将报错：`cannot reflectively create enum objects`。

## IOC容器

### 容器的载体以及容器的加载

需要保证容器是单例，可以使用上面讲到的枚举类来实现，避免被反射和序列化破坏 。

在初始化时，容器将所有被特定注解（如`@Component, @Controller`等）标注的类进行初始化构建，用一个map来存储，其中，键为`Class<?>`，值为`Object`，通过class对象可以获取对应类的实例。

定义一个`loadBean`的方法，传入包名，扫描包下的所有类，如果该类被特定注解标记，那么通过反射获取它的无参构造函数，`newInstance`出一个实例放入map中。

```java
@Slf4j
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public class BeanContainer {

    /**
     * 存放所有被配置标记的目标对象的map
     */
    private final Map<Class<?>, Object> beanMap = new ConcurrentHashMap<>();

    /**
     * 加载bean的注解列表
     */
    private static final List<Class<? extends Annotation>> BEAN_ANNOTATION =
            Arrays.asList(Component.class, Controller.class, Service.class, Repository.class);

    /**
     * 获取bean容器实例
     *
     * @return
     */
    public static BeanContainer getInstance() {
        return ContainerHolder.HOLDER.instance;
    }


    private enum ContainerHolder {
        HOLDER;
        private BeanContainer instance;

        ContainerHolder() {
            instance = new BeanContainer();
        }
    }

    /**
     * 容器是否被加载过
     */
    private boolean loaded = false;

    /**
     * 容器是否已经被加载过
     *
     * @return 容器是否已经被加载过
     */
    public boolean isLoaded() {
        return loaded;
    }

    /**
     * Bean实例数量
     *
     * @return 数量
     */
    public int size() {
        return beanMap.size();
    }

    /**
     * 扫描加载所有的bean
     *
     * @param packageName 包名
     */
    public synchronized void loadBeans(String packageName) {
        //判断bean容器是否被加载过
        if (isLoaded()) {
            log.warn("BeanContainer has been loaded");
            return;
        }
        Set<Class<?>> classSet = ClassUtil.extractPackageClass(packageName);
        if (ValidationUtil.isEmpty(classSet)) {
            log.warn("extract nothing from packageName " + packageName);
            return;
        }
        for (Class<?> clazz : classSet) {
            for (Class<? extends Annotation> annotation : BEAN_ANNOTATION) {
                //如果类上面标记了定义的注解
                if (clazz.isAnnotationPresent(annotation)) {
                    //将目标类本身作为键，目标类的实例作为值，放入到beanMap中
                    beanMap.put(clazz, ClassUtil.newInstance(clazz, true));
                }
            }
        }
        loaded = true;
    }
}
```

### 提供容器对外操作的方法

```java
/**
 * 添加一个class对象及其Bean实例
 *
 * @param clazz class对象
 * @param bean  bean实例
 * @return 原有的bean实例，没有则返回null
 */
public Object addBean(Class<?> clazz, Object bean) {
    return beanMap.put(clazz, bean);
}

/**
 * 移除一个IOC容器管理的对象
 *
 * @param clazz class对象
 * @return 删除的bean实例，没有则返回null
 */
public Object removeBean(Class<?> clazz) {
    return beanMap.remove(clazz);
}

/**
 * 根据class对象获取bean实例
 *
 * @param clazz class对象
 * @return bean实例
 */
public Object getBean(Class<?> clazz) {
    return beanMap.get(clazz);
}

/**
 * 获取容器管理的所有class对象集合
 *
 * @return class集合
 */
public Set<Class<?>> getClasses() {
    return beanMap.keySet();
}

/**
 * 获取所有bean的集合
 *
 * @return bean集合
 */
public Set<Object> getBeans() {
    return new HashSet<>(beanMap.values());
}

/**
 * 根据注解筛选出bean的class集合
 *
 * @param annotation 注解
 * @return class集合
 */
public Set<Class<?>> getClassesByAnnotation(Class<? extends Annotation> annotation) {
    //1、获取beanMap的所有class对象
    Set<Class<?>> keySet = getClasses();
    if (ValidationUtil.isEmpty(keySet)) {
        log.warn("nothing in beanMap");
        return null;
    }
    Set<Class<?>> classSet = new HashSet<>();
    //2、通过注解筛选出被注解标记的class对象，并添加到classSet中
    for (Class<?> clazz : keySet) {
        if (clazz.isAnnotationPresent(annotation)) {
            classSet.add(clazz);
        }
    }
    return classSet.size() > 0 ? classSet : null;
}

/**
 * 通过接口或者父类获取实现类或者子类的class集合，不包括其本身
 *
 * @param interfaceOrClass 接口class或者父类class
 * @return class集合
 */
public Set<Class<?>> getClassesBySuper(Class<?> interfaceOrClass) {
    //1、获取beanMap的所有class对象
    Set<Class<?>> keySet = getClasses();
    if (ValidationUtil.isEmpty(keySet)) {
        log.warn("nothing in beanMap");
        return null;
    }
    Set<Class<?>> classSet = new HashSet<>();
    //2、判断keySet里的元素是否是传入的接口或者类的子类，如果是，就将其添加到classSet中
    for (Class<?> clazz : keySet) {
        //注意，isAssignableFrom还包括自己本身，需要排除在外
        if (interfaceOrClass.isAssignableFrom(clazz) && !clazz.equals(interfaceOrClass)) {
            classSet.add(clazz);
        }
    }
    return classSet.size() > 0 ? classSet : null;
}
```

以下是测试类：

```java
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
public class BeanContainerTest {
    private static BeanContainer beanContainer;

    @BeforeAll
    static void init() {
        beanContainer = BeanContainer.getInstance();
    }

    @DisplayName("加载目标类及其实例到BeanContainer：loadBeansTest")
    @Order(1)
    @Test
    public void loadBeansTest() {
        Assertions.assertEquals(false, beanContainer.isLoaded());
        beanContainer.loadBeans("com.yikang");
        Assertions.assertEquals(7, beanContainer.size());
        Assertions.assertEquals(true, beanContainer.isLoaded());
    }

    @DisplayName("根据获取其实例：getBeanTest")
    @Order(2)
    @Test
    public void getBean() {
        MainPageController mainPageController = (MainPageController) beanContainer.getBean(MainPageController.class);
        Assertions.assertEquals(true, mainPageController instanceof MainPageController);
        DispatcherServlet dispatcherServlet = (DispatcherServlet) beanContainer.getBean(DispatcherServlet.class);
        Assertions.assertEquals(null, dispatcherServlet);
    }

    @DisplayName("根据注解获取对应的实例：getClassesByAnnotationTest")
    @Order(3)
    @Test
    public void getClassesByAnnotationTest() {
        Assertions.assertEquals(true, beanContainer.isLoaded());
        Assertions.assertEquals(3, beanContainer.getClassesByAnnotation(Controller.class).size());
    }

    @DisplayName("根据接口获取实现类：getClassesBySuperTest")
    @Order(4)
    @Test
    public void getClassesBySuperTest() {
        Assertions.assertEquals(true, beanContainer.isLoaded());
        Assertions.assertEquals(true, beanContainer.getClassesBySuper(HeadLineService.class).contains(HeadLineServiceImpl.class));
    }
}
```

### 实现容器的依赖注入

上述实现是不够的，可以注意到，在一些类的实现中可能会注入其他类，例如`@Autowired`注解，我们在进行实例化某个类时还需要将它需要的类依赖注入到其中。

首先需要定义相关注解标签，实现创建被注解标记的成员变量实例，并将其注入到成员变量里。

首先定义`@Autowired`注解：

```java
/* 
* Autowired 目前仅支持成员变量的注入
*/
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Autowired {
    String value() default "";
}
```

其中`value`变量用于指定注入的实具体现类（如果有多个实现类的话）。

然后编写注入的代码：

大致的流程是这样的：

1、遍历Bean容器中所有的class对象

2、遍历class对象的所有成员变量

3、找出被Autowired标记的成员变量

4、获取这些成员变量的类型即class

5、获取这些成员变量的类型在容器里对应的实例

6、通过反射将对应的成员变量实例注入到成员变量所在类的实例里

之前我们已经编写了BeanContainer类中的getClassesBySuper的方法来获取某个父类或容器的所有实现，可以调用该方法来获取所有的实现，然后根据`@Autowired`的成员变量`value`来具体地选择具体的实现类。

```java
@Slf4j
public class DependencyInjector {
    /**
     * bean容器
     */
    private BeanContainer beanContainer;

    public DependencyInjector() {
        beanContainer = BeanContainer.getInstance();
    }

    /**
     * 执行Ioc
     */
    public void doIoc() {
        //1、遍历Bean容器中所有的class对象
        Set<Class<?>> classSet = beanContainer.getClasses();
        if (ValidationUtil.isEmpty(classSet)) {
            log.warn("empty classSet in BeanContainer");
            return;
        }
        for (Class<?> clazz : classSet) {
            //2、遍历class对象的所有成员变量
            Field[] fields = clazz.getDeclaredFields();
            if (ValidationUtil.isEmpty(fields)) {
                continue;
            }
            for (Field field : fields) {
                //3、找出被Autowired标记的成员变量
                if (field.isAnnotationPresent(Autowired.class)) {
                    Autowired autowired = field.getAnnotation(Autowired.class);
                    String autowiredValue = autowired.value();
                    //4、获取这些成员变量的类型
                    Class<?> fieldClass = field.getType();
                    //5、获取这些成员变量的类型在容器里对应的实例
                    Object fieldValue = getFieldInstance(fieldClass, autowiredValue);
                    if (fieldValue == null) {
                        throw new RuntimeException("unable to inject relevant type, target fieldClass is:" + fieldClass.getName() + " " + autowiredValue);
                    } else {
                        //6、通过反射将对应的成员变量实例注入到成员变量所在类的实例里
                        Object targetBean = beanContainer.getBean(clazz);
                        ClassUtil.setField(field, targetBean, fieldValue, true);
                    }
                }
            }
        }
    }

    /**
     * 根据class在beanContainer里获取其实例或者实现类
     *
     * @param fieldClass     class
     * @param autowiredValue 如果实现类有多个，那么指定注入哪种实现类
     * @return class实例
     */
    private Object getFieldInstance(Class<?> fieldClass, String autowiredValue) {
        Object fieldValue = beanContainer.getBean(fieldClass);
        if (fieldValue != null) {
            return fieldValue;
        } else {
            //fieldClass可能是接口的接口
            Class<?> implementedClass = getImplementClass(fieldClass, autowiredValue);
            if (implementedClass != null) {
                return beanContainer.getBean(implementedClass);
            } else {
                return null;
            }
        }
    }

    /**
     * 获取接口的实现类
     *
     * @param fieldClass     class
     * @param autowiredValue 如果实现类有多个，那么指定注入哪种实现类
     * @return 实现类
     */
    private Class<?> getImplementClass(Class<?> fieldClass, String autowiredValue) {
        Set<Class<?>> classSet = beanContainer.getClassesBySuper(fieldClass);
        if (!ValidationUtil.isEmpty(classSet)) {
            if (ValidationUtil.isEmpty(autowiredValue)) {
                if (classSet.size() == 1) {
                    return classSet.iterator().next();
                } else {
                    //如果有多个实现类，但是用户没有指定哪种，则抛出异常
                    throw new RuntimeException("multiple implemented classes for" + fieldClass.getName() + "please set @Autowired's value to pick one");
                }
            } else {
                for (Class<?> clazz : classSet) {
                    if (autowiredValue.equals(clazz.getSimpleName())) {
                        return clazz;
                    }
                }
            }
        }
        return null;
    }

}
```

以下为测试类：

```java
public class DependencyInjectorTest {

    @DisplayName("依赖注入doIoc")
    @Test
    public void doIocTest() {
        BeanContainer beanContainer = BeanContainer.getInstance();
        beanContainer.loadBeans("com.yikang");
        Assertions.assertEquals(true, beanContainer.isLoaded());
        MainPageController mainPageController = (MainPageController) beanContainer.getBean(MainPageController.class);
        Assertions.assertEquals(true, mainPageController instanceof MainPageController);
        Assertions.assertEquals(null, mainPageController.getHeadLineShopCategoryCombineService());
        new DependencyInjector().doIoc();
        Assertions.assertNotEquals(null, mainPageController.getHeadLineShopCategoryCombineService());
        Assertions.assertEquals(true, mainPageController.getHeadLineShopCategoryCombineService() instanceof HeadLineShopCategoryCombineServiceImpl);
    }
}
```

## SpringAOP

### CGLIB动态代理

+ 不要求被代理类实现接口
+ 内部主要封装了ASM Java字节码操控框架
+ 动态生成子类以覆盖非final的方法，绑定钩子回调自定义拦截器

### JDK动态代理和CGLIB

#### 实现机制

+ JDK动态代理：基于反射机制实现，要求业务必须实现接口
+ CGLIB：基于ASM机制实现，生成业务类的子类作为代理类

#### JDK动态代理的优势

+ JDK原生，在JVM里运行较为可靠
+ 平滑支持JDK版本的升级

#### CGLIB

+ 被代理对象无需实现接口，能实现代理类的无侵入

#### SpringAOP的低层机制

+ CGLIB和JDK动态代理共存
+ 默认策略：Bean实现了接口则用JDK，否则使用CGLIB

## 实现AOP1.0

### AOP 1.0总览

使用CGLIB来实现：不需要业务类实现接口，相对灵活

+ 解决标记的问题，定义横切逻辑的骨架
+ 定义Aspect横切逻辑以及被代理方法的执行顺序
+ 将横切逻辑织入到被代理的对象以生成动态代理对象

### 解决横切逻辑的标记问题以及定义Aspect骨架

+ 定义与横切逻辑相关的注解
+ 定义供外部使用的横切逻辑的骨架

首先定义与横切逻辑相关的注解`@Aspect`, `@Order`：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Aspect {
    /**
     * 需要被织入横切逻辑的注解标签
     */
    Class<? extends Annotation> value();
}
```

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Order {
    /**
     * 控制类的执行顺序，值越小优先级越高
     */
    int value();
}
```

定义供外部使用的横切逻辑的骨架。

这里不实现`@Before`等注解，反之用继承的方式来实现`before`，`afterReturning`，`afterThrowing`等方法，如果需要实现切面，那么需要继承这个抽象类：

```java
public abstract class DefaultAspect {
    /**
     * 钩子方法，用于事前拦截
     *
     * @param targetClass 被代理的目标类
     * @param method      被代理的目标方法
     * @param args        被代理的目标方法对应的参数列表
     * @throws Throwable
     */
    public void before(Class<?> targetClass, Method method, Object[] args) throws Throwable {

    }

    /**
     * 钩子方法，用于事后拦截
     *
     * @param targetClass 被代理的目标类
     * @param method      被代理的目标方法
     * @param args        被代理的目标方法对应的参数列表
     * @param returnValue 被代理的目标方法执行后的返回值
     * @return 被目标方法执行后的返回值
     * @throws Throwable
     */
    public Object afterReturning(Class<?> targetClass, Method method, Object[] args, Object returnValue) throws Throwable {
        return returnValue;
    }

    /**
     * @param targetClass 被代理的目标类
     * @param method      被代理的目标方法
     * @param args        被代理的目标方法对应的参数列表
     * @param e           被代理的目标方法抛出的异常
     * @throws Throwable
     */
    public void afterThrowing(Class<?> targetClass, Method method, Object[] args, Throwable e) throws Throwable {

    }
}
```

### 实现Aspect横切逻辑以及被代理方法的定序执行

+ 创建MethodInterceptor的实现类
+ 定义必要的成员变量——被代理的类以及Aspect的列表
+ 按照Order对Aspect进行排序
+ 实现对横切逻辑以及被代理对象方法的定序执行

一个aspect有三个属性，需要织入横切逻辑的对象，order，对应的aspect事件逻辑（before、after等）。

将“需要织入横切逻辑的对象”保存在MethodInterceptor实现类中，并且MethodInterceptor实现类中保存某对象被织入的所有aspect。

将“order”、“aspect事件逻辑”合并到一个类`AspectInfo`中：

```java
@AllArgsConstructor
@Getter
public class AspectInfo {
    private int orderIndex;
    private DefaultAspect aspectObject;
}
```

重点是MethodInterceptor的实现类，该实现类某个对象拥有的所有Aspect的引用列表，在重写的`intercept`方法中的`methodProxy.invokeSuper`前后分别调用相应的before与after方法：

```java
public class AspectListExecutor implements MethodInterceptor {
    //被代理的类
    private Class<?> targetClass;

    //排好序的列表
    @Getter
    List<AspectInfo> sortedAspectInfoList;

    public AspectListExecutor(Class<?> targetClass, List<AspectInfo> aspectInfoList) {
        this.targetClass = targetClass;
        this.sortedAspectInfoList = sortAspectInfoList(aspectInfoList);
    }

    /**
     * 按照order的值进行升序排序，确保order值小的aspect先被织入
     *
     * @param aspectInfoList aspectInfo列表
     * @return 排好序后的aspectInfo列表
     */
    private List<AspectInfo> sortAspectInfoList(List<AspectInfo> aspectInfoList) {
        Collections.sort(aspectInfoList, Comparator.comparingInt(AspectInfo::getOrderIndex));
        return aspectInfoList;
    }

    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        Object returnValue = null;
        if (ValidationUtil.isEmpty(sortedAspectInfoList)) return returnValue;
        //1、按照order的顺序升级执行完所有Aspect的before方法
        invokeBeforeAdvices(method, args);
        try {
            //2、执行被代理类的方法
            returnValue = methodProxy.invokeSuper(o, args);
            //3、如果被代理方法正常返回，则按照order的顺序降序执行完所有Aspect的afterReturning方法
            invokeAfterReturningAdvices(method, args, returnValue);
        } catch (Exception e) {
            //4、如果被代理方法抛出异常，则按照order的顺序降序执行完所有Aspect的afterThrowing方法
            invokeAfterThrowingAdvices(method, args, e);
        }
        return returnValue;
    }

    //4、如果被代理方法抛出异常，则按照order的顺序降序执行完所有Aspect的afterThrowing方法
    private void invokeAfterThrowingAdvices(Method method, Object[] args, Exception e) throws Throwable {
        for (int i = sortedAspectInfoList.size() - 1; i >= 0; i--) {
            sortedAspectInfoList.get(i).getAspectObject().afterThrowing(targetClass, method, args, e);
        }
    }

    //3、如果被代理方法正常返回，则按照order的顺序降序执行完所有Aspect的afterReturning方法
    private void invokeAfterReturningAdvices(Method method, Object[] args, Object returnValue) throws Throwable {
        for (int i = sortedAspectInfoList.size() - 1; i >= 0; i--) {
            sortedAspectInfoList.get(i).getAspectObject().afterReturning(targetClass, method, args, returnValue);
        }
    }

    //1、按照order的顺序升级执行完所有Aspect的before方法
    private void invokeBeforeAdvices(Method method, Object[] args) throws Throwable {
        for (AspectInfo aspectInfo : sortedAspectInfoList) {
            aspectInfo.getAspectObject().before(targetClass, method, args);
        }
    }
}
```

然后创建工具类，用于创建动态代理对象返回：

```java
public class ProxyCreator {
    /**
     * 创建动态代理对象并返回
     *
     * @param targetClass       被代理的Class对象
     * @param methodInterceptor 方法拦截器
     * @return 代理对象
     */
    public static Object createProxy(Class<?> targetClass, MethodInterceptor methodInterceptor) {
        return Enhancer.create(targetClass, methodInterceptor);
    }
}
```

### 将横切逻辑织入到被代理的对象以生成动态代理对象

大致的逻辑为：使用`BeanContainer`获取所有的切面类（即被`@Aspect`标注的bean），因为多个切面类可以织入同一个对象，因此需要维护一个map对象，通过某个对象可以获取它的织入切面列表。通过某个对象以及它的织入切面列表可以构造出MethodInterceptor实现类，进而构造出动态代理对象，将动态代理对象替换未被代理前的类实例放入到`BeanContainer`中。

```java
public class AspectWeaver {
    private BeanContainer beanContainer;

    public AspectWeaver() {
        beanContainer = BeanContainer.getInstance();
    }

    public void doAop() {
        //1、获取所有的切面类
        Set<Class<?>> aspectSet = beanContainer.getClassesByAnnotation(Aspect.class);
        //2、将切面类按照不同的织入目标进行切分
        //实现的是简化版的，只能对注解标注的对象织入切面
        Map<Class<? extends Annotation>, List<AspectInfo>> categorizedMap = new HashMap<>();
        if (ValidationUtil.isEmpty(aspectSet)) return;
        for (Class<?> aspectClass : aspectSet) {
            if (verifyAspect(aspectClass)) {
                categorizedAspect(categorizedMap, aspectClass);
            } else {
                throw new RuntimeException("@Aspect and @Order have not been added to the Aspect class," +
                        "or Aspect class does not extend from DefaultAspect, or the value in Aspect Tag equals @Aspect");
            }
        }
        //3、按照不同的织入目标分别去按序织入aspect的逻辑
        if (ValidationUtil.isEmpty(categorizedMap)) return;
        for (Class<? extends Annotation> category : categorizedMap.keySet()) {
            weaveByCategory(category, categorizedMap.get(category));
        }
    }

    private void weaveByCategory(Class<? extends Annotation> category, List<AspectInfo> aspectInfoList) {
        //1、获取被代理类的集合
        Set<Class<?>> classSet = beanContainer.getClassesByAnnotation(category);
        if (ValidationUtil.isEmpty(classSet)) return;
        //2、遍历被代理类，分别为每个被代理类生成动态代理实例
        for (Class<?> targetClass : classSet) {
            //创建动态代理对象
            AspectListExecutor executor = new AspectListExecutor(targetClass, aspectInfoList);
            Object proxyBean = ProxyCreator.createProxy(targetClass, executor);
            //3、将动态代理对象实例添加到容器里，取代未被代理前的类实例
            beanContainer.addBean(targetClass, proxyBean);
        }
    }

    /**
     * 将切面类按照不同的织入目标进行切分
     *
     * @param categorizedMap 分类map
     * @param aspectClass    aspect对象
     */
    private void categorizedAspect(Map<Class<? extends Annotation>, List<AspectInfo>> categorizedMap, Class<?> aspectClass) {
        Order orderTag = aspectClass.getAnnotation(Order.class);
        Aspect aspectTag = aspectClass.getAnnotation(Aspect.class);
        DefaultAspect aspect = (DefaultAspect) beanContainer.getBean(aspectClass);
        AspectInfo aspectInfo = new AspectInfo(orderTag.value(), aspect);
        if (!categorizedMap.containsKey(aspectTag.value())) {
            //如果织入的join point第一次出现，则以该join point为key，以新创建的List<AspectInfo>为value
            List<AspectInfo> aspectInfoList = new ArrayList<>();
            aspectInfoList.add(aspectInfo);
            categorizedMap.put(aspectTag.value(), aspectInfoList);
        } else {
            //如果织入的join point不是第一次出现，则往join point对应的value里添加新的aspectInfo
            categorizedMap.get(aspectTag.value()).add(aspectInfo);
        }
    }

    /**
     * 框架中一定要遵守给Aspect类添加@Aspect和@Order标签的规范，同时，必须继承自DefaultAspect.class
     * 此外，@Aspect的属性值不能是他本身
     *
     * @param aspectClass 需要验证的Aspect对象
     * @return
     */
    private boolean verifyAspect(Class<?> aspectClass) {
        return aspectClass.isAnnotationPresent(Aspect.class)
                && aspectClass.isAnnotationPresent(Order.class)
                && DefaultAspect.class.isAssignableFrom(aspectClass)
                && aspectClass.getAnnotation(Aspect.class).value() != Aspect.class;
    }
}
```

测试代码如下：

注意，应该先进行切面织入再进行依赖注入。

```java
public class AspectWeaverTest {
    @DisplayName("织入通用逻辑测试：doAop")
    @Test
    public void doAopTest() {
        BeanContainer beanContainer = BeanContainer.getInstance();
        beanContainer.loadBeans("com.yikang");
        new AspectWeaver().doAop();
        new DependencyInjector().doIoc();
        HeadLineOperationController headLineOperationController = (HeadLineOperationController) beanContainer.getBean(HeadLineOperationController.class);
        headLineOperationController.addHeadLine(null, null);
    }
}
```

### 待改进的地方

+ Aspect只支持对被某个标签标记的类进行横切逻辑的织入
+ 需要借助AspectJ来达到上述目的

## AspectJ框架

提供了完整的AOP解决方案，是AOP的Java实现版本

+ 定义切面语法以及切面语法的解析机制
+ 提供了强大的织入工具

AspectJ框架的织入时机：静态织入和LTW

+ 编译时织入：利用ajc，将切面逻辑织入到类里生成class文件（静态织入）
+ 编译后织入：利用ajc，修改javac编译出来的class文件（静态织入）
+ 类加载期织入：利用java agent，在类加载的时候织入切面逻辑（动态织入）

对于SpringAOP2.0，仅仅用到了AspectJ的切面语法，并没有使用ajc编译工具

+ 避免增加用户的学习成本
+ 只是默认不使用，如果想用ajc还是可以引入的
+ 织入机制沿用自己的CGLIB和JDK动态代理机制

## 折衷方案改进框架里的AOP

之前的pointcut并不灵活，只支持对被某个标签标记的类进行横切逻辑的织入，因此需要引入AspectJ的切面表达式和相关的定位解析机制。

使用最小的改造成本，换取尽可能大的收益：

+ 让Pointcut更灵活
+ 调研结果：只需要引入AspectJ的切面表达式和相关的定位解析机制

首先在pom文件中引入依赖：

```xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.5</version>
</dependency>
```

创建`PointcutLocator`，用于解析AspectJ表达式并且定位被织入的目标，每个aspect都有自己的一个`PointcutLocator`，对每个class对象/method，可以调用当前aspect对应的pointcutLocator.roughMatch/accurateMatch来判断是否是当前aspect织入的对象。

```java
/**
 * 解析Aspect表达式并且定位被织入的目标，与aspect一一对应
 */
public class PointcutLocator {
    /**
     * Pointcut解析器，直接给它赋值上AspectJ的所有表达式，以便支持对众多表达式的解析
     */
    private PointcutParser pointcutParser = PointcutParser.getPointcutParserSupportingSpecifiedPrimitivesAndUsingContextClassloaderForResolution(
            PointcutParser.getAllSupportedPointcutPrimitives()
    );
    /**
     * 表达式解析器
     */
    private PointcutExpression pointcutExpression;

    public PointcutLocator(String expression) {
        this.pointcutExpression = pointcutParser.parsePointcutExpression(expression);
    }

    /**
     * 判断传入的Class对象是否是Aspect的目标代理类，即匹配Pointcut表达式（初筛）
     * 例如，需要去匹配class a，则需要这样调用：
     * pointcutLocator.roughMatches(a)
     *
     * @param targetClass 目标类
     * @return 是否匹配
     */
    public boolean roughMatches(Class<?> targetClass) {
        //couldMatchJoinPointsInType只能校验within
        //不能校验(execution, call, get, set)，面对无法校验的表达式，直接返回true
        return pointcutExpression.couldMatchJoinPointsInType(targetClass);
    }

    /**
     * 判断传入的Method对象是否是AspectJ的目标代理方法，即匹配Pointcut表达式（精筛）
     *
     * @param method
     * @return
     */
    public boolean accurateMatches(Method method) {
        ShadowMatch shadowMatch = pointcutExpression.matchesMethodExecution(method);
        return shadowMatch.alwaysMatches();
    }
}
```

在之前，`@Aspect`的`value`类型为`Class<? extend Annotation>`，只能作用于被注解标注的类，现在引入了AspectJ，作用的范围会更广，因此需要改成`String`类型：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Aspect {
    String pointcut();
}
```

同时，每个aspect都有着自己的PointcutLocator，这是用于识别织入对象的工具，因此需要将PointcutLocator整合进`AspectInfo`中：

```java
@AllArgsConstructor
@Getter
public class AspectInfo {
    private int orderIndex;
    private DefaultAspect aspectObject;
    private PointcutLocator pointcutLocator;
}
```

筛选织入对象分为两个阶段，一是初筛，调用的是`PointcurLocator#roughMatches`方法，放在`AspectWeaver`中调用（`roughMatches`只需要接受`Class`参数，在`AspectWeaver`中可以满足条件）；二是精筛，调用的是`PointcutLocator#accurateMatches`，需要传入`Method`参数，在`AspectWeaver`中不太方便，需要遍历`Method`，因此放在MethodInterceptor的实现类中调用。

`AspectWeaver`：

```java
public class AspectWeaver {
    private BeanContainer beanContainer;

    public AspectWeaver() {
        beanContainer = BeanContainer.getInstance();
    }

    public void doAop() {
        //1、获取所有的切面类
        Set<Class<?>> aspectSet = beanContainer.getClassesByAnnotation(Aspect.class);
        if (ValidationUtil.isEmpty(aspectSet)) return;
        //2、拼装AspectInfoList
        List<AspectInfo> aspectInfoList = packAspectInfoList(aspectSet);
        //3、遍历容器里的类
        Set<Class<?>> classSet = beanContainer.getClasses();
        for (Class<?> targetClass : classSet) {
            //排除AspectClass自身
            if (targetClass.isAnnotationPresent(Aspect.class)) {
                continue;
            }
            //4、粗筛符合条件的Aspect
            List<AspectInfo> roughMatchedAspectList = collectRoughMatchedAspectListForSpecificClass(aspectInfoList, targetClass);
            //5、尝试进行Aspect的织入
            wrapIfNecessary(roughMatchedAspectList, targetClass);
        }

    }

    private void wrapIfNecessary(List<AspectInfo> roughMatchedAspectList, Class<?> targetClass) {
        if (ValidationUtil.isEmpty(roughMatchedAspectList)) return;
        //创建动态代理对象
        AspectListExecutor aspectListExecutor = new AspectListExecutor(targetClass, roughMatchedAspectList);
        Object proxyBean = ProxyCreator.createProxy(targetClass, aspectListExecutor);
        beanContainer.addBean(targetClass, proxyBean);
    }

    private List<AspectInfo> collectRoughMatchedAspectListForSpecificClass(List<AspectInfo> aspectInfoList, Class<?> targetClass) {
        List<AspectInfo> roughMatchedAspectList = new ArrayList<>();
        for (AspectInfo aspectInfo : aspectInfoList) {
            //初筛
            if (aspectInfo.getPointcutLocator().roughMatches(targetClass)) {
                roughMatchedAspectList.add(aspectInfo);
            }
        }
        return roughMatchedAspectList;
    }

    private List<AspectInfo> packAspectInfoList(Set<Class<?>> aspectSet) {
        List<AspectInfo> aspectInfoList = new ArrayList<>();
        for (Class<?> aspectClass : aspectSet) {
            if (verifyAspect(aspectClass)) {
                Order orderTag = aspectClass.getAnnotation(Order.class);
                Aspect aspectTag = aspectClass.getAnnotation(Aspect.class);
                DefaultAspect defaultAspect = (DefaultAspect) beanContainer.getBean(aspectClass);
                //初始化表达式定位器
                PointcutLocator pointcutLocator = new PointcutLocator(aspectTag.pointcut());
                AspectInfo aspectInfo = new AspectInfo(orderTag.value(), defaultAspect, pointcutLocator);
                aspectInfoList.add(aspectInfo);
            } else {
                throw new RuntimeException("@Aspect and @Order must be added to the Aspect class," +
                        " and Aspect class must extend from DefaultAspect");
            }
        }
        return aspectInfoList;
    }

    /**
     * 框架中一定要遵守给Aspect类添加@Aspect和@Order标签的规范，同时，必须继承自DefaultAspect.class
     * 此外，@Aspect的属性值不能是他本身
     *
     * @param aspectClass 需要验证的Aspect对象
     * @return
     */
    private boolean verifyAspect(Class<?> aspectClass) {
        return aspectClass.isAnnotationPresent(Aspect.class)
                && aspectClass.isAnnotationPresent(Order.class)
                && DefaultAspect.class.isAssignableFrom(aspectClass);
    }
}
```

MethodInterceptor实现类：

```java
package org.simpleframework.aop;

import lombok.Getter;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
import org.simpleframework.aop.aspect.AspectInfo;
import org.simpleframework.util.ValidationUtil;

import java.lang.reflect.Method;
import java.util.Collections;
import java.util.Comparator;
import java.util.Iterator;
import java.util.List;

public class AspectListExecutor implements MethodInterceptor {
    //被代理的类
    private Class<?> targetClass;

    //排好序的列表
    @Getter
    List<AspectInfo> sortedAspectInfoList;

    public AspectListExecutor(Class<?> targetClass, List<AspectInfo> aspectInfoList) {
        this.targetClass = targetClass;
        this.sortedAspectInfoList = sortAspectInfoList(aspectInfoList);
    }

    /**
     * 按照order的值进行升序排序，确保order值小的aspect先被织入
     *
     * @param aspectInfoList aspectInfo列表
     * @return 排好序后的aspectInfo列表
     */
    private List<AspectInfo> sortAspectInfoList(List<AspectInfo> aspectInfoList) {
        Collections.sort(aspectInfoList, Comparator.comparingInt(AspectInfo::getOrderIndex));
        return aspectInfoList;
    }

    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        Object returnValue = null;
        collectAccurateMatchedAspectList(method);
        if (ValidationUtil.isEmpty(sortedAspectInfoList)) {
            //这里要注意，如果没有匹配的切面，那么还是需要调用原方法
            returnValue = methodProxy.invokeSuper(o, args);
            return returnValue;
        }
        //1、按照order的顺序升级执行完所有Aspect的before方法
        invokeBeforeAdvices(method, args);
        try {
            //2、执行被代理类的方法
            returnValue = methodProxy.invokeSuper(o, args);
            //3、如果被代理方法正常返回，则按照order的顺序降序执行完所有Aspect的afterReturning方法
            invokeAfterReturningAdvices(method, args, returnValue);
        } catch (Exception e) {
            //4、如果被代理方法抛出异常，则按照order的顺序降序执行完所有Aspect的afterThrowing方法
            invokeAfterThrowingAdvices(method, args, e);
        }
        return returnValue;
    }

    private void collectAccurateMatchedAspectList(Method method) {
        if (ValidationUtil.isEmpty(sortedAspectInfoList)) return;
        Iterator<AspectInfo> it = sortedAspectInfoList.iterator();
        while (it.hasNext()) {
            AspectInfo aspectInfo = it.next();
            if (!aspectInfo.getPointcutLocator().accurateMatches(method)) {
                it.remove();
            }
        }
    }

    //4、如果被代理方法抛出异常，则按照order的顺序降序执行完所有Aspect的afterThrowing方法
    private void invokeAfterThrowingAdvices(Method method, Object[] args, Exception e) throws Throwable {
        for (int i = sortedAspectInfoList.size() - 1; i >= 0; i--) {
            sortedAspectInfoList.get(i).getAspectObject().afterThrowing(targetClass, method, args, e);
        }
    }

    //3、如果被代理方法正常返回，则按照order的顺序降序执行完所有Aspect的afterReturning方法
    private void invokeAfterReturningAdvices(Method method, Object[] args, Object returnValue) throws Throwable {
        for (int i = sortedAspectInfoList.size() - 1; i >= 0; i--) {
            sortedAspectInfoList.get(i).getAspectObject().afterReturning(targetClass, method, args, returnValue);
        }
    }

    //1、按照order的顺序升级执行完所有Aspect的before方法
    private void invokeBeforeAdvices(Method method, Object[] args) throws Throwable {
        for (AspectInfo aspectInfo : sortedAspectInfoList) {
            aspectInfo.getAspectObject().before(targetClass, method, args);
        }
    }
}
```

这样就可以使用与Spring框架相同的AspectJ表达式了。

## 小tip

可以设置VM参数来更好地研究源码：

```bash
# 将中间生成的代理类保存下来
-Djdk.proxy.ProxyGenerator.saveGeneratedFiles=true
# 将加载的类打印出来
-XX:+TraceClassLoading
```



