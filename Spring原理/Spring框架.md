​Spring是一个开源框架，是为了解决企业应用开发的复杂性而创建的。 

​目前是JavaEE开发的灵魂框架，可以简化JavaEE开发，非常方便整合其他框架，无侵入地进行功能增强。

​Spring的核心就是控制反转(IoC)和面向切面(AOP) 。

# 软件模块化

方便分工合作，不同的人开发不同模块/在不同的时间开发不同模块
不同层次用不同技术实现，每一层都可以独立测试，不同层次部署到的服务器也可以不同(提高系统的整体性能)

**最简单的划分**

* 视图层（View）
  用于实现界面，负责界面长啥样，界面上的数据如何输入输出
  面向功能的设计原则
  用VUE，微信小程序...实现
  部署到Web服务器上，只存储静态网页和静态图片
* 控制器层（Controller）
  用于视图层和模型层之间的相互转换
  用SpringMVC实现
* 模型层（Model）
  用于实现逻辑和数据存储
  面向对象的设计原则
  部署在应用服务器（Tomcat，Netty...）

**对模型层进行更进一步划分**

更细致的划分使得系统结构更加简单
不同层次用不同技术实现，每一层都可以独立测试
* 服务层（Service）
  负责业务逻辑
  用Spring容器Bean对象实现
* 数据访问层（DAO）
  负责获取数据组装成对象
  用MySql，Redis...实现
* 映射层（Mapper）
  负责数据对象模型向数据库的关系模型转换
  用MyBatis实现
* 实体层（Entity）
  数据的载体
  不依赖于任何框架技术，就是POJO对象

# Spring技术栈

## Servlet技术栈（常用）

阻塞式的Servlet技术栈利用Servlet实现基于Web的服务器端的应用。服务器端针对每一个请求开一个独立的线程处理，结果返回给前端

**阻塞式是什么意思**

CPU和内存是有限的，不可能无限接受请求开辟线程，切换线程，所以一般会设置一个线程池来控制请求的个数。每一个请求都会包括一些慢速操作（比如io操作），这会让当前线程处于等待状态，正是这些处于阻塞状态的线程和线程池的存在，使得新的请求无法被马上处理（即使CPU此时的工作量并不大），导致CPU资源利用率低

**使用的技术框架**

* SpringMVC实现控制层（响应及回应HTTP请求）
* 逻辑层运用JDBC/JPA...访问数据库

**Servlet原理**

## 响应式技术栈
将整个任务分解成若干小任务。接受到请求后，不是简单地用一个线程来处理
**事件驱动思想**
把慢速操作（比如io操作）置于一个事件队列来侦听是否完成了这个操作。这些慢速操作不会独占一个线程，也就不会使有限的资源被慢速操作阻塞。当慢速操作执行完后，才会开一个线程去完成那些高速操作
**使用的技术框架**
* Spring WebFlux实现控制器层
* 不再用Tomcat服务器，而是用像Netty这样的高速异步服务器
* 访问数据层的技术也与Servlet技术栈不同

io操作较频繁的系统（如：微服务的网关）中使用响应式技术栈

# Bean对象

**配置Bean对象最基本要求：**

  1. 告诉Spring这是一个Bean对象
  2.  给Bean对象起个名字（唯一）
  3. 设置这个Bean对象它活多长

**Bean对象的作用域Scope**

>**Singleton:容器中只有这一个Bean对象**
Prototype：每次使用都会产生一个Bean对象
Request：每个Http请求产生一个Bean对象
Session：每个Session产生一个Bean对象
Application：在ServletContext范围内产生唯一Baen对象

实际上，在需求量爆炸的今天，只有`Singleton`还有使用的意义，后三个都有创建太多对象导致服务器裂开的隐患，最后一个是因为现在极少在Servlet里写代码了，所以也不用了。

**告诉Spring这是一个Bean对象+起名**

* 用xml配置：已经被淘汰了，不需要
* 用注解：`@Component`+`@ComponentScan`，首字母小写的类名是Bean对象的名字
* 用代码：在有`@Configuration`的类里创建带`@Bean`的方法，方法的返回值是Bean对象的类型，方法名（首字母小写）是Bean对象的名字

没啥特别要求就用注解（简单），有一些比较灵活的操作或特殊要求就用代码

**Bean对象的生命周期**

Bean级生命周期接口：
>`BeanNameAware`：对象实例化并设置属性之后调用其setBeanName(String beanName)可以设置BeanName
`BeanFactoryAware`：调用setBeanName之后调用其setBeanFactory (BeanFactory var1)可以设置BeanFactory
`ApplicationContextAware`：调用其setApplicationContext(ApplicationContext context) 获得ApplicationContext
`IntializingBean`:属性赋值完成之后调用afterPropertiesSet() 
`DisposableBean`:关闭容器时调用 destroy() 

容器级生命周期接口：InstantiationAwareBeanPostProcessorAdapter和BeanPostProcessor

在属性构造之前和构造之后都会调用容器级接口

![](image/Bean%E5%AF%B9%E8%B1%A1%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

# IoC和DI

## 理论
高度耦合的例子：类A的一个属性是类B
>Inversion of Control“控制反转”：我对于我关联的对象没有控制权，我是交给别人去控制（比如IoC容器）
Dependency Injection“依赖注入”：我交出控制权后，别人要告诉我，我跟谁依赖（和谁关联对象）
>
>实际上，IoC和DI就是在做一个事情的两个方面，一个交权，一个告诉我交权后谁来管，也可以说DI是实现IoC的一种方法

交出控制权后建立耦合一般有两种方法（外界来告诉我我应该和谁关联）
1. 构造函数传一个B进来关联
2. `set`函数

**注**：出现循环依赖时（A对象关联B对象，B对象关联C对象，C对象关联A对象），只能用`set`函数和`@Lazy`建立耦合。循环依赖在设计的时候应该尽可能地避免

应用的例子：
1. **做切片测试（IoC的最大收益是-松耦合，使得切片测试成为可能）**：比如类A有一个是属性类B，测试代码的时候，因为耦合关联的原因，势必会同时测试A和B的代码，出错了还要找是A错了还是B错了。而通过IoC（或者说DI）注入的是一个无代码的假B，测试的时候就只会测试A的代码，这就是切片测试
2. 我们可以将对象交给IoC容器进行管理，比如当我们需要一个接口的实现时，由它根据配置文件来决定到底给我们哪一个实现类，这样，我们就可以不用再关心我们要去使用哪一个实现类了，我们只需要关心，给到我的一定是一个可以正常使用的实现类，能用就完事了，反正接口定义了啥，我只管调，这样，我们就可以放心地让一个人去写视图层的代码，一个人去写业务层的代码。
 
***

所有对象都是由Spring容器来创建，我们写的业务代码称为业务对象，也就是Bean对象（或POJO对象）

在Spring容器中，Bean对象间的依赖不用程序员去管理，而是通过*配置信息*告诉Spring容器Bean对象之间的依赖。

**可以用XML文件配置，但现在常用的是用注解进行开发**

## 注解实现IoC 

>@Component, @Controller, @Service ,@Repository	

​上述4个注解都是加到类上的。

他们都可以起到类似bean标签的作用，可以把加了该注解的类的对象放入Spring容器中。

实际使用时选择任意一个都可以。但是后3个注解是语义化注解。

如果是Service类（实现类），要求使用@Service。

如果是Dao类要求使用@Repository

​如果是Controllerl类要求使用@Controller(或@RestController)

如果是其他类可以使用@Component


## 注解实现DI

一个bean已经放入Spring容器中了，则可以使用下列注解实现属性注入，让Spring容器帮我们完成属性的赋值。

>@Resource和@Autowired机制上的不同：
@Resource默认ByName如果找不到则ByType，可以添加到set方法、字段上。
>
>@Autowired默认是byType，只会根据类型寻找，可以添加在构造方法、set方法、字段、方法参数上。
>
>因为@Resource的匹配机制更加合理高效，因此官方并不推荐使用@Autowired字段注入。
当然，实际上Spring官方更推荐使用基于构造方法或是Setter的@Autowired注入，比如Setter 注入的一个好处是，Setter 方法使该类的对象能够在以后重新配置或重新注入。

**用`@Resource`注入**

注解给属性（实际上就是构造函数）
```java
@Component
public class Student {
    String name;
    int age;
    @Resource
    Card card;
}
```

注解给`set`方法，将自动注入`set`函数的参数
```java 
@Component
public class Student {
    String name;
    int age;
    Card card;
    @Resource
    public void setCard(Card card) {
        System.out.println("我是通过set方法注入的属性");
        this.card = card;
    }
}
```

**用`@Autowire`注入**
默认匹配规则：
Spring容器首先查询对应类型的Bean对象，如果只有一个，则直接装配给该属性/该`set`方法的参数；如果同类型的Bean对象不止一个，则以变量名称作为id查找同类对象

注意：
1. `@Autowire`不支持标注在一个属性上（违背封装性，只是为了方便）
2. `@Autowire`推荐标注在构造方法上，不推荐标注在set方法上
3. `@Autowire`也可以标注在`@Bean`方法上

## 注解替代XML配置文件

涉及的注解
>@Configuration，@ComponentScan，@Bean，@PropertySource

**如何选择**
①SSM  
自己项目中的类的IOC和DI都使用注解。第三方jar包中的类，配置组件扫描时使用xml进行配置。

②SpringBoot：纯注解开发

***

**@Configuration**
标注在类上，表示当前类是一个配置类。用注解类来完全替换掉xml配置文件。
注意：如果使用配置类替换了xml配置，spring容器要使用：AnnotationConfigApplicationContext

***
**@ComponentScan**
加在配置类上，可以用来代替context:component-scan标签来配置组件扫描。
basePackages属性来指定要扫描的包。

~~~~java
@Configuration
@ComponentScan(basePackages = "com.xxx")//指定要扫描的包
public class ApplicationConfig {
}
~~~~

***
**@Bean**
可以用来代替bean标签，主要**用于第三方类的注入**。（别人写的类在jar包里，我们修改不了代码，也就不能用@Component之类的注解实现IoC）【比如连接池的配置...】
使用：定义一个方法，在方法中创建对应的对象并且作为返回值返回。然后在方法上加上@Bean注解，注解的value属性来设置bean的名称。

~~~~java
@Configuration
@ComponentScan(basePackages = "com.xxx")
public class ApplicationConfig {

    @Bean("dataSource")
    public DruidDataSource getDataSource(){
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setDriverClassName("com.mysql.jdbc.Driver");
        druidDataSource.setUsername("root");
        druidDataSource.setUrl("jdbc:mysql://localhost:3306/mybatis_db");
        druidDataSource.setPassword("root");
        return druidDataSource;
    }

}
~~~~

**注意事项：如果同一种类型的对象在容器中只有一个，那么可以不设置bean的名称。**

~~~~java
@Configuration
@ComponentScan(basePackages = "com.sangeng")
public class ApplicationConfig {

    @Bean
    public DruidDataSource getDataSource(){
        同上
    }
}
~~~~

获取方式如下：

~~~~java
public static void main(String[] args) {
    //创建注解容器
    AnnotationConfigApplicationContext app = new AnnotationConfigApplicationContext(ApplicationConfig.class);
    //根据对应类的字节码对象获取
    DataSource bean = app.getBean(DataSource.class);
    System.out.println(userService);
}
~~~~

***
**@PropertySource**
可以用来代替context:property-placeholder，让Spring读取指定的properties文件。然后可以使用@Value来获取读取到的值。
使用：在配置类上加@PropertySource注解，注解的value属性来设置properties文件的路径。然后在配置类中定义成员变量。在成员变量上使用@Value注解来获取读到的值并给对应的成员变量赋值。

例如：
~~~~properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis_db
jdbc.username=root
jdbc.password=root
~~~~

读取文件并且获取值

~~~~java
@Configuration
@ComponentScan(basePackages = "com.xxx")
@PropertySource("jdbc.properties")
public class ApplicationConfig {

    @Value("${jdbc.driver}")
    private String driverClassName;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;


    @Bean
    public DruidDataSource getDataSource(){
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setDriverClassName(driverClassName);
        druidDataSource.setUsername(username);
        druidDataSource.setUrl(url);
        druidDataSource.setPassword(password);
        return druidDataSource;
    }
}
~~~~

**注意事项：使用@Value获取读到的properties文件中的值时使用的是${key},而不是#{key}。**


## Spring容器

Spring容器依靠BeanFactory创建Bean对象并管理

>BeanFactory的工作
>* 读Bean对象配置信息
>* Bean对象的加载和实例化
>* 维护Bean对象之间的依赖关系
>* Bean对象的生命周期

现在更常用的是ApplicatoinContext，它是BeanFactory的一个子接口，在其基础上又拓展了许多功能，可以视为强化版的BeanFactory

# 面向切面编程AOP

## 理论

**首要解决的问题**
有很多相同的代码段（比如抛异常）散落在许多不同地方（以Controller层为例），我们希望通过剔除掉那些代码得到一个纯净的Controller层（只做这一层该做的事，抛异常那些操作虽然是必要的，但我不想把它参杂在这一层里）

把那些嵌在Controller里面的琐碎零散的点称为`Join Point`，将这些点对应的代码写成一个`Advice`，然后用一个定义好的`Point Cut`让它在编译或执行的时候切到那些点上去

如果是在编译时切进来，则要引入一个新的编程语言，而Spring框架采用的是**执行时切进来**，这样的话就只能切在执行前或执行后，所以Spring的AOP是阉割版的面向切面编程

Spring的AOP思想实际上就是：在运行时，动态地将代码切入到类的指定方法、指定位置上。也就是说，我们可以使用AOP来帮助我们在方法执行前或执行之后，做一些额外的操作，实际上，就是代理！

## Spring如何实现AOP

**Point Cut**

用来定义这些代码是切在哪些位置

>**arg()**
**@args()**
**execution:**  Join Point方法签名满足特定条件
**@annotation：**   Join Point方法用特定注解标注（比较灵活，不需要切入点有什么共同特征，想要切入的地方打个标签就行）
**within**
**@within**
**target**
**@target**

**Advice**

用来写代码的，就是说明切上去之后要干嘛

>@Before：定义在Join Point运行前要干的事
@After：定义在Join Point运行后要干的事（无论执行是否成功）
@AfterReturning：定义在Join Point运行成功后要干的事
@AfterThrowing：定义在Join Point运行失败，抛出异常后要干的事
@Around：定义在Join Point运行前和后要干的事

# SpringMVC

控制器不需要实现任何接口，它是个POJO，支持RestFul风格
DispatcherServlet是SpringMVC的中枢

# SpringBoot

**SpringBoot的作用的是方便开发独立的Spring应用程序**

* 内嵌Tomcat、Jetty或Undertow：
  在这之前，Spring打包后是一个war包，需要在服务器上装一个Tomcat把war包部署上去才能用。而SprignBoot内嵌Tomcat后，直接就可以打包成jar包部署到服务器上运行
* 采用Starter POM简化Maven的配置
* 大量采用约定简化Spring的配置
* 提供产品级的运行监控功能

**SpringBoot配置途径**

* java虚拟机
* application.propetites和application.yml文件
* java命令行
* 定义环境变量

以上的所有配置的参数都可以通过DI的方式（`@Value(属性名称)`将Bean对象的某个属性设置成从配置里读进来的某个参数）用到所有的Bean对象里

![](image/SpringBoot%E9%85%8D%E7%BD%AE.png)

# Spring的缓存机制