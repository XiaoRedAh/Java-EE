# 从理论分析Spring原理

## 软件模块化

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

## Spring技术栈

### Servlet技术栈（常用）

阻塞式的Servlet技术栈利用Servlet实现基于Web的服务器端的应用。服务器端针对每一个请求开一个独立的线程处理，结果返回给前端

**阻塞式是什么意思**

CPU和内存是有限的，不可能无限接受请求开辟线程，切换线程，所以一般会设置一个线程池来控制请求的个数。每一个请求都会包括一些慢速操作（比如io操作），这会让当前线程处于等待状态，正是这些处于阻塞状态的线程和线程池的存在，使得新的请求无法被马上处理（即使CPU此时的工作量并不大），导致CPU资源利用率低

**使用的技术框架**

* SpringMVC实现控制层（响应及回应HTTP请求）
* 逻辑层运用JDBC/JPA...访问数据库

**Servlet原理**



### 响应式技术栈
将整个任务分解成若干小任务。接受到请求后，不是简单地用一个线程来处理
**事件驱动思想**
把慢速操作（比如io操作）置于一个事件队列来侦听是否完成了这个操作。这些慢速操作不会独占一个线程，也就不会使有限的资源被慢速操作阻塞。当慢速操作执行完后，才会开一个线程去完成那些高速操作
**使用的技术框架**
* Spring WebFlux实现控制器层
* 不再用Tomcat服务器，而是用像Netty这样的高速异步服务器
* 访问数据层的技术也与Servlet技术栈不同

io操作较频繁的系统（如：微服务的网关）中使用响应式技术栈

## Spring Bean对象

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

## IoC，DI
### 理论
高度耦合的例子：类A的一个属性是类B
>Inversion of Control“控制反转”：我对于我关联的对象没有控制权，我是交给别人去控制（比如IoC容器）
Dependency Injection“依赖注入”：我交出控制权后，别人要告诉我，我跟谁依赖（和谁关联对象）
实际上，IoC和DI就是在做一个事情的两个方面，一个交权，一个告诉我交权后谁来管，也可以说DI是实现IoC的一种方法

交出控制权后建立耦合一般有两种方法（外界来告诉我我应该和谁关联）
1. 构造函数传一个B进来关联
2. `set`函数

**注**：出现循环依赖时（A对象关联B对象，B对象关联C对象，C对象关联A对象），只能用`set`函数和`@Lazy`建立耦合。循环依赖在设计的时候应该尽可能地避免

应用的例子：
1. **做切片测试（IoC的最大收益是-松耦合，使得切片测试成为可能）**：比如类A有一个是属性类B，测试代码的时候，因为耦合关联的原因，势必会同时测试A和B的代码，出错了还要找是A错了还是B错了。而通过IoC（或者说DI）注入的是一个无代码的假B，测试的时候就只会测试A的代码，这就是切片测试
2. 我们可以将对象交给IoC容器进行管理，比如当我们需要一个接口的实现时，由它根据配置文件来决定到底给我们哪一个实现类，这样，我们就可以不用再关心我们要去使用哪一个实现类了，我们只需要关心，给到我的一定是一个可以正常使用的实现类，能用就完事了，反正接口定义了啥，我只管调，这样，我们就可以放心地让一个人去写视图层的代码，一个人去写业务层的代码。
 
### Spring如何实现IoC和DI的
所有对象都是由Spring容器来创建，我们写的业务代码称为业务对象，也就是Bean对象（或POJO对象）

在Spring容器中，Bean对象间的依赖不用程序员去管理，而是通过*配置信息*告诉Spring容器Bean对象之间的依赖。

现在常用的是用注解进行依赖注入。

**用`@Resource`注入**
* 注解给属性（实际上就是构造函数）
```java
@Component
public class Student {
    String name;
    int age;
    @Resource
    Card card;
}
```
* 注解给`set`方法，将自动注入`set`函数的参数
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

## Spring容器

Spring容器依靠BeanFactory创建Bean对象并管理

>BeanFactory的工作
>* 读Bean对象配置信息
>* Bean对象的加载和实例化
>* 维护Bean对象之间的依赖关系
>* Bean对象的生命周期

现在更常用的是ApplicatoinContext，它是BeanFactory的一个子接口，在其基础上又拓展了许多功能，可以视为强化版的BeanFactory

## 面向切面编程AOP
### 理论
**首要解决的问题**：
有很多相同的代码段（比如抛异常）散落在许多不同地方（以Controller层为例），我们希望通过剔除掉那些代码得到一个纯净的Controller层（只做这一层该做的事，抛异常那些操作虽然是必要的，但我不想把它参杂在这一层里）

把那些嵌在Controller里面的琐碎零散的点称为`Join Point`，将这些点对应的代码写成一个`Advice`，然后用一个定义好的`Point Cut`让它在编译或执行的时候切到那些点上去

如果是在编译时切进来，则要引入一个新的编程语言，而Spring框架采用的是**执行时切进来**，这样的话就只能切在执行前或执行后，所以Spring的AOP是阉割版的面向切面编程

Spring的AOP思想实际上就是：在运行时，动态地将代码切入到类的指定方法、指定位置上。也就是说，我们可以使用AOP来帮助我们在方法执行前或执行之后，做一些额外的操作，实际上，就是代理！

### Spring如何实现AOP

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

## SpringMVC
控制器不需要实现任何接口，它是个POJO，支持RestFul风格
DispatcherServlet是SpringMVC的中枢

## SpringBoot

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



## Spring的缓存机制

# 结合源码探究Spring原理

## BeanFactory接口与IoC容器

首先大致了解一下ApplicationContext的加载流程：

![img](https://upload-images.jianshu.io/upload_images/749633-e707e5a84194d13a.png?imageMogr2/auto-orient/strip|imageView2/2/w/852)

整个过程极为复杂，就从`ApplicationContext`开始吧

由于Spring的源码极为复杂，因此不可能再像了解其他框架那样直接自底向上逐行干源码了，而是先从一些最基本的接口定义开始，自顶向下逐步瓦解。

`ApplicationContext`最顶层接口：一直往上，会发现有一个`AbstractApplicationContext`类，右键生成一个UML类图查看：

![image-20211209202503121](/Users/nagocoler/Library/Application Support/typora-user-images/image-20211209202503121.png)

发现最顶层实际上是一个`BeanFactory`接口,此接口中定义了很多的行为：

```java
public interface BeanFactory {
    String FACTORY_BEAN_PREFIX = "&";

    Object getBean(String var1) throws BeansException;

    <T> T getBean(String var1, Class<T> var2) throws BeansException;

    Object getBean(String var1, Object... var2) throws BeansException;

    <T> T getBean(Class<T> var1) throws BeansException;

    <T> T getBean(Class<T> var1, Object... var2) throws BeansException;

    <T> ObjectProvider<T> getBeanProvider(Class<T> var1);

    <T> ObjectProvider<T> getBeanProvider(ResolvableType var1);

    boolean containsBean(String var1);

    boolean isSingleton(String var1) throws NoSuchBeanDefinitionException;

    boolean isPrototype(String var1) throws NoSuchBeanDefinitionException;

    boolean isTypeMatch(String var1, ResolvableType var2) throws NoSuchBeanDefinitionException;

    boolean isTypeMatch(String var1, Class<?> var2) throws NoSuchBeanDefinitionException;

    @Nullable
    Class<?> getType(String var1) throws NoSuchBeanDefinitionException;

    @Nullable
    Class<?> getType(String var1, boolean var2) throws NoSuchBeanDefinitionException;

    String[] getAliases(String var1);
}
```

其中最眼熟的就是`getBean()`方法了，此方法被重载了很多次，可以接受多种参数，一个IoC容器最基本的行为在此接口中已经被定义好了。也就是说，所有的`BeanFactory`实现类都应该具备容器管理Bean的基本能力。

直接找到此接口的一个抽象实现`AbstractBeanFactory`类，它实现了`getBean()`方法：

```java
public Object getBean(String name) throws BeansException {
    return this.doGetBean(name, (Class)null, (Object[])null, false);
}
```

看看`doGetBean()`里面干了什么：

因为所有的Bean默认都是单例模式，对象只会存在一个，因此它会先调用父类的`getSingleton()`方法来直接获取单例对象。如果有的话，就可以直接拿到Bean的实例。

```java
protected <T> T doGetBean(String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly) throws BeansException {
    String beanName = this.transformedBeanName(name);
    Object sharedInstance = this.getSingleton(beanName);
    Object beanInstance;
    if (sharedInstance != null && args == null) {
      ...
```

如果没有单例对象，就会进入else代码块，首先会进行一个判断：

这是为了解决循环依赖进行的处理：比如A和B都是以原型模式进行创建，而A中需要注入B，B中需要注入A，这时就会出现A还未创建完成，就需要B，而B这时也没创建完成，因为B需要A，而A等着B，这样就只能无限循环下去了。
```java
if (this.isPrototypeCurrentlyInCreation(beanName)) {
    throw new BeanCurrentlyInCreationException(beanName);
}
```

![img](https://pica.zhimg.com/80/v2-a426f1c897402e708f3781581211008f_1440w.jpg?source=1940ef5c)

而在单例模式下，由于每个Bean只会创建一个实例，Spring完全有机会处理好循环依赖的问题，只需要一个正确的赋值操作实现循环即可。

回到`getSingleton()`方法中，看看单例模式是如何解决循环依赖问题的：

```java
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    Object singletonObject = this.singletonObjects.get(beanName);
  	//先从第一层列表中拿Bean实例，拿到直接返回
    if (singletonObject == null && this.isSingletonCurrentlyInCreation(beanName)) {
      	//第一层拿不到，并且已经认定为处于循环状态，看看第二层有没有
        singletonObject = this.earlySingletonObjects.get(beanName);
        if (singletonObject == null && allowEarlyReference) {
            synchronized(this.singletonObjects) {
              	//加锁再执行一次上述流程
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) {
                    singletonObject = this.earlySingletonObjects.get(beanName);
                    if (singletonObject == null) {
                      	//仍然没有获取到实例，只能从singletonFactory中获取了
                        ObjectFactory<?> singletonFactory = (ObjectFactory)this.singletonFactories.get(beanName);
                        if (singletonFactory != null) {
                            singletonObject = singletonFactory.getObject();
                          	//丢进earlySingletonObjects中，下次就可以直接在第二层拿到了
                            this.earlySingletonObjects.put(beanName, singletonObject);
                            this.singletonFactories.remove(beanName);
                        }
                    }
                }
            }
        }
    }

    return singletonObject;
}
```

实际上它使用了三层列表的方式来处理循环依赖的问题。包括：

* singletonObjects
* earlySingletonObjects
* singletonFactories

当第一层拿不到时，会接着判断这个Bean是否处于创建状态`isSingletonCurrentlyInCreation()`。它会从一个Set集合中查询，这个集合中存储了已经创建但还未注入属性的实例对象（也就是正在创建），如果说发现此Bean处于正在创建状态（一定是因为某个Bean需要注入这个Bean的实例），就可以断定它应该是出现了循环依赖的情况。

earlySingletonObjects相当于是专门处理循环依赖的表，一般包含singletonObjects中的全部实例，如果这个里面还是没有，接着往下走，这时会从singletonFactories中获取（所有的Bean初始化完成之后都会被丢进singletonFactories，也就是只创建了，但是还没进行依赖注入的时候）在获取到后，向earlySingletonObjects中丢入此Bean的实例，并将实例从singletonFactories中移除。

梳理一下流程：以“A依赖于B，B依赖于A”为例：

1. 假如A先载入，那么A首先进入了singletonFactories中，这时还没进行依赖注入，A中的B还是null
   * singletonFactories：A
   * earlySingletonObjects：
   * singletonObjects：
2. 接着注入A的依赖B了，但是B还没初始化，因此现在先把B给载入了，B构造完成后也进了singletonFactories
   * singletonFactories：A，B
   * earlySingletonObjects：
   * singletonObjects：
3. 接着为B注入依赖，发现B依赖于A，这时又得去获取A的实例，根据上面的分析，这时候A还在singletonFactories中，那么它会被丢进earlySingletonObjects，并且从singletonFactories中移除，然后返回A的实例（注意此时A中的B依赖还是null）
   * singletonFactories：B
   * earlySingletonObjects：A
   * singletonObjects：
4. 这时B已经完成依赖注入了，因此可以直接丢进singletonObjects中
   * singletonFactories：
   * earlySingletonObjects：A
   * singletonObjects：B
5. 然后再将B注入到A中，完成A的依赖注入，A也被丢进singletonObjects中，至此循环依赖完成，A和B完成实例创建
   * singletonFactories：
   * earlySingletonObjects：
   * singletonObjects：A，B

回到之前的地方（继续`doGetBean()`），原型模式下如果出现循环依赖会直接抛出异常（因为会一直new，解决不了），如果不存在会接着向下：

BeanFactory会先看当前是否存在Bean的定义，如果没有会直接用父BeanFactory各种找。这里出现了一个新的接口`BeanDefinition`：Bean工厂需要拿到Bean的信息才可以去生成这个Bean的实例对象，而`BeanDefinition`就是用于存放Bean的信息的，Bean信息正是从XML配置文件读取或是注解扫描后得到的。


```java
//BeanFactory存在父子关系
BeanFactory parentBeanFactory = this.getParentBeanFactory();
//如果存在父BeanFactory，同时当前BeanFactory没有这个Bean的定义
if (parentBeanFactory != null && !this.containsBeanDefinition(beanName)) {
  	//这里是因为Bean可能有别名，找最原始的那个名称
    String nameToLookup = this.originalBeanName(name);
    if (parentBeanFactory instanceof AbstractBeanFactory) {
      	//向父BeanFactory递归查找
        return ((AbstractBeanFactory)parentBeanFactory).doGetBean(nameToLookup, requiredType, args, typeCheckOnly);
    }

    if (args != null) {
      	//根据参数查找
        return parentBeanFactory.getBean(nameToLookup, args);
    }

    if (requiredType != null) {
      	//根据类型查找
        return parentBeanFactory.getBean(nameToLookup, requiredType);
    }

  	//各种找
    return parentBeanFactory.getBean(nameToLookup);
}
```

如果此BeanFactory不存在父BeanFactory或是包含了Bean的定义，那么会接着往下走，这时只能自己创建Bean了，首先会拿到一个`RootBeanDefinition`对象：

```java
try {
    if (requiredType != null) {
        beanCreation.tag("beanType", requiredType::toString);
    }

    RootBeanDefinition mbd = this.getMergedLocalBeanDefinition(beanName);
```

下面的内容就非常复杂了，但是它一定是根据对应的类型（单例、原型）进行了对应的处理，最后自行创建一个新的对象返回。一个Bean的加载流程为：

首先拿到`BeanDefinition`定义，选择对应的构造方法，通过反射进行实例化，然后进行属性填充（依赖注入），完成之后再调用初始化方法（init-method），最后如果存在AOP，则会生成一个代理对象，最后返回的才是真正得到的Bean对象。

最后回到`ApplicationContext`中，实际上，它就是一个强化版的`BeanFactory`，在最基本的Bean管理基础上，还添加了：

* 国际化（MessageSource）
* 访问资源，如URL和文件（ResourceLoader）
* 载入多个（有继承关系）上下文 
* 消息发送、响应机制（ApplicationEventPublisher）
* AOP机制

构造方法中都会调用`refresh()`方法来刷新应用程序上下文：

```java
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
    this();
    this.register(componentClasses);
    this.refresh();
}
```

## PostProcessor与AOP原理

`PostProcessor`其实是Spring提供的一种后置处理机制，它可以让我们能够插手Bean、BeanFactory、BeanDefinition的创建过程，相当于进行一个最终的处理，而最后得到的结果（比如Bean实例、Bean定义等）就是经过后置处理器返回的结果，它是整个加载过程的最后一步。

AOP机制正是通过它来实现的。

首先来认识一下接口`BeanPostProcessor`，它相当于Bean初始化的一个后置动作，可以直接实现此接口：

```java
//注意后置处理器也要进行注册
@Component
public class TestBeanProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println(beanName);  //打印bean的名称
        return bean;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return BeanPostProcessor.super.postProcessBeforeInitialization(bean, beanName);
    }
}
```

此接口中包括两个方法，一个是`postProcessAfterInitialization`用于在Bean初始化之后进行处理，还有一个`postProcessBeforeInitialization`用于在Bean初始化之前进行处理

*注意这里的初始化不是创建对象，而是调用类的初始化方法（此时对象已经完成依赖注入了）*
比如：

```java
@Component
public class TestBeanProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("我是之后："+beanName);
        return bean;   //这里返回的Bean相当于最终的结果了，我们依然能够插手修改，这里返回之后是什么就是什么了
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("我是之前："+beanName);
        return bean;   //这里返回的Bean会交给下一个阶段，也就是初始化方法
    }
}
```

```java
@Component
public class TestServiceImpl implements TestService{

    public TestServiceImpl(){
        System.out.println("我是构造方法");
    }

    @PostConstruct
    public void init(){
        System.out.println("我是初始化方法");
    }

    TestMapper mapper;

    @Autowired
    public void setMapper(TestMapper mapper) {
        System.out.println("我是依赖注入");
        this.mapper = mapper;
    }
  	
  	...
```

而TestServiceImpl的加载顺序为：

```xml
我是构造方法
我是依赖注入
我是之前：testServiceImpl
我是初始化方法
我是之后：testServiceImpl
```

总结一下Bean的加载流程：

1. [Bean定义]首先扫描Bean，加载Bean定义
2.  [创建Bean]根据Bean定义通过反射创建Bean实例
3.  [依赖注入]进行依赖注入（顺便解决循环依赖问题）
4.  [初始化Bean]BeanPostProcessor的初始化之前方法`postProcessBeforeInitialization`
5.  [初始化Bean]Bean初始化方法
6.  [初始化Bean]BeanPostProcessor的初始化之后方法`postProcessAfterInitialization`
7.  [完成]最终得到的Bean加载完成的实例

AOP实际上也是通过这种机制实现的，它的实现类是`AnnotationAwareAspectJAutoProxyCreator`，而它就是在最后对Bean进行了代理，因此最后得到的结果实际上就是一个动态代理的对象（有关详细实现过程，以后再继续深入）

`TestBeanProcessor`这个类没有被注册，那按理说它不应该参与到Bean的初始化流程中的，为什么它直接就被加载了呢？

看看`@EnableAspectJAutoProxy`如何定义的：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({AspectJAutoProxyRegistrar.class})
public @interface EnableAspectJAutoProxy {
    boolean proxyTargetClass() default false;

    boolean exposeProxy() default false;
}
```

我们发现它使用了`@Import`来注册`AspectJAutoProxyRegistrar`，那么这个类又是什么呢

```java
class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {
    AspectJAutoProxyRegistrar() {
    }

    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
      	//注册AnnotationAwareAspectJAutoProxyCreator到容器中
        AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);
        AnnotationAttributes enableAspectJAutoProxy = AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
        if (enableAspectJAutoProxy != null) {
            if (enableAspectJAutoProxy.getBoolean("proxyTargetClass")) {
                AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
            }

            if (enableAspectJAutoProxy.getBoolean("exposeProxy")) {
                AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
            }
        }

    }
}
```

它实现了接口`ImportBeanDefinitionRegistrar`，这个接口是Spring提供的一种Bean加载机制，它支持直接向容器中添加Bean定义，容器也会加载这个Bean：

* ImportBeanDefinitionRegistrar类只能通过其他类@Import的方式来加载，通常是启动类或配置类。
* 使用@Import，如果括号中的类是ImportBeanDefinitionRegistrar的实现类，则会调用接口中方法（一般用于注册Bean）
* 实现该接口的类拥有注册bean的能力。(此接口提供了一个`BeanDefinitionRegistry`正是用于注册Bean的定义的)

因此，打上了`@EnableAspectJAutoProxy`注解之后，首先会通过`@Import`加载AspectJAutoProxyRegistrar，然后调用其`registerBeanDefinitions`方法，然后使用工具类注册AnnotationAwareAspectJAutoProxyCreator到容器中，这样在每个Bean创建之后，如果需要使用AOP，那么就会通过AOP的后置处理器进行处理，最后返回一个代理对象。

尝试编写一个自己的ImportBeanDefinitionRegistrar实现，首先编写一个测试Bean：

```java
public class TestBean {
    
    @PostConstruct
    void init(){
        System.out.println("我被初始化了！");
    }
}
```

```java
public class TestBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        BeanDefinition definition = BeanDefinitionBuilder.rootBeanDefinition(Student.class).getBeanDefinition();
        registry.registerBeanDefinition("lbwnb", definition);
    }
}
```

观察控制台输出，成功加载Bean实例。

与`BeanPostProcessor`差不多的还有`BeanFactoryPostProcessor`，它和前者一样，也是用于我们自己处理后置动作的，不过这里是用于处理BeanFactory加载的后置动作，`BeanDefinitionRegistryPostProcessor`直接继承自`BeanFactoryPostProcessor`，并且还添加了新的动作`postProcessBeanDefinitionRegistry`，可以在这里动态添加Bean定义或是修改已经存在的Bean定义

这里直接演示`BeanDefinitionRegistryPostProcessor`的实现：

```java
@Component
public class TestDefinitionProcessor implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        System.out.println("我是Bean定义后置处理！");
        BeanDefinition definition = BeanDefinitionBuilder.rootBeanDefinition(TestBean.class).getBeanDefinition();
        registry.registerBeanDefinition("lbwnb", definition);
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        System.out.println("我是Bean工厂后置处理！");
    }
}
```

在这里注册Bean定义其实和之前那种方法效果一样。

最后，再完善一下Bean加载流程（加粗部分是新增的）：

[Bean定义]首先扫描Bean，加载Bean定义 -> **[Bean定义]Bean定义和Bean工厂后置处理** -> [创建Bean]根据Bean定义通过反射创建Bean实例 -> [依赖注入]进行依赖注入（顺便解决循环依赖问题）-> [初始化Bean]BeanPostProcessor的初始化之前方法 -> [初始化Bean]Bean初始化方法 -> [初始化Bean]BeanPostProcessor的初始化之后方法 -> [完成]最终得到的Bean加载完成的实例

最后再来研究一下ApplicationContext中的`refresh()`方法：

```java
public void refresh() throws BeansException, IllegalStateException {
    synchronized(this.startupShutdownMonitor) {
        StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");
        this.prepareRefresh();
        ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
      	//初始化Bean工厂
        this.prepareBeanFactory(beanFactory);

        try {
            this.postProcessBeanFactory(beanFactory);
            StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
          	//调用所有的Bean工厂、Bean注册后置处理器
            this.invokeBeanFactoryPostProcessors(beanFactory);
          	//注册Bean后置处理器（包括Spring内部的）
            this.registerBeanPostProcessors(beanFactory);
            beanPostProcess.end();
          	//国际化支持
            this.initMessageSource();
          	//监听和事件广播
            this.initApplicationEventMulticaster();
            this.onRefresh();
            this.registerListeners();
          	//实例化所有的Bean
            this.finishBeanFactoryInitialization(beanFactory);
            this.finishRefresh();
        } catch (BeansException var10) {
            if (this.logger.isWarnEnabled()) {
                this.logger.warn("Exception encountered during context initialization - cancelling refresh attempt: " + var10);
            }

            this.destroyBeans();
            this.cancelRefresh(var10);
            throw var10;
        } finally {
            this.resetCommonCaches();
            contextRefresh.end();
        }

    }
}
```

## `@MapperScan`实现Mybatis整合的原理

来看Mybatis的`@MapperScan`是如何实现的，现在理解起来就非常简单了。

打开查看：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({MapperScannerRegistrar.class})
@Repeatable(MapperScans.class)
public @interface MapperScan {
    String[] value() default {};

    String[] basePackages() default {};
  	...
```

和Aop一样，它也是通过Registrar机制，通过`@Import`来进行Bean的注册，现在来看看`MapperScannerRegistrar`是个什么东西，关键代码如下：

```java
void registerBeanDefinitions(AnnotationMetadata annoMeta, AnnotationAttributes annoAttrs, BeanDefinitionRegistry registry, String beanName) {
    BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(MapperScannerConfigurer.class);
    builder.addPropertyValue("processPropertyPlaceHolders", true);
    Class<? extends Annotation> annotationClass = annoAttrs.getClass("annotationClass");
    if (!Annotation.class.equals(annotationClass)) {
        builder.addPropertyValue("annotationClass", annotationClass);
    }

    Class<?> markerInterface = annoAttrs.getClass("markerInterface");
    if (!Class.class.equals(markerInterface)) {
        builder.addPropertyValue("markerInterface", markerInterface);
    }

    Class<? extends BeanNameGenerator> generatorClass = annoAttrs.getClass("nameGenerator");
    if (!BeanNameGenerator.class.equals(generatorClass)) {
        builder.addPropertyValue("nameGenerator", BeanUtils.instantiateClass(generatorClass));
    }

    Class<? extends MapperFactoryBean> mapperFactoryBeanClass = annoAttrs.getClass("factoryBean");
    if (!MapperFactoryBean.class.equals(mapperFactoryBeanClass)) {
        builder.addPropertyValue("mapperFactoryBeanClass", mapperFactoryBeanClass);
    }

    String sqlSessionTemplateRef = annoAttrs.getString("sqlSessionTemplateRef");
    if (StringUtils.hasText(sqlSessionTemplateRef)) {
        builder.addPropertyValue("sqlSessionTemplateBeanName", annoAttrs.getString("sqlSessionTemplateRef"));
    }

    String sqlSessionFactoryRef = annoAttrs.getString("sqlSessionFactoryRef");
    if (StringUtils.hasText(sqlSessionFactoryRef)) {
        builder.addPropertyValue("sqlSessionFactoryBeanName", annoAttrs.getString("sqlSessionFactoryRef"));
    }

    List<String> basePackages = new ArrayList();
    basePackages.addAll((Collection)Arrays.stream(annoAttrs.getStringArray("value")).filter(StringUtils::hasText).collect(Collectors.toList()));
    basePackages.addAll((Collection)Arrays.stream(annoAttrs.getStringArray("basePackages")).filter(StringUtils::hasText).collect(Collectors.toList()));
    basePackages.addAll((Collection)Arrays.stream(annoAttrs.getClassArray("basePackageClasses")).map(ClassUtils::getPackageName).collect(Collectors.toList()));
    if (basePackages.isEmpty()) {
        basePackages.add(getDefaultBasePackage(annoMeta));
    }

    String lazyInitialization = annoAttrs.getString("lazyInitialization");
    if (StringUtils.hasText(lazyInitialization)) {
        builder.addPropertyValue("lazyInitialization", lazyInitialization);
    }

    String defaultScope = annoAttrs.getString("defaultScope");
    if (!"".equals(defaultScope)) {
        builder.addPropertyValue("defaultScope", defaultScope);
    }

    builder.addPropertyValue("basePackage", StringUtils.collectionToCommaDelimitedString(basePackages));
    registry.registerBeanDefinition(beanName, builder.getBeanDefinition());
}
```

虽然很长很多，但是这些代码都是在添加一些Bean定义的属性，而最关键的则是最上方的`MapperScannerConfigurer`，Mybatis将其Bean信息注册到了容器中

它实现了BeanDefinitionRegistryPostProcessor，也就是说它为Bean信息加载提供了后置处理

```java
public class MapperScannerConfigurer implements BeanDefinitionRegistryPostProcessor, InitializingBean, ApplicationContextAware, BeanNameAware {
    private String basePackage;
```

接着来看看它在Bean信息后置处理中做了什么：

```java
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
    if (this.processPropertyPlaceHolders) {
        this.processPropertyPlaceHolders();
    }

  	//初始化类路径Mapper扫描器，它相当于是一个工具类，可以快速扫描出整个包下的类定义信息
  	//ClassPathMapperScanner是Mybatis自己实现的一个扫描器，修改了一些扫描规则
    ClassPathMapperScanner scanner = new ClassPathMapperScanner(registry);
    scanner.setAddToConfig(this.addToConfig);
    scanner.setAnnotationClass(this.annotationClass);
    scanner.setMarkerInterface(this.markerInterface);
    scanner.setSqlSessionFactory(this.sqlSessionFactory);
    scanner.setSqlSessionTemplate(this.sqlSessionTemplate);
    scanner.setSqlSessionFactoryBeanName(this.sqlSessionFactoryBeanName);
    scanner.setSqlSessionTemplateBeanName(this.sqlSessionTemplateBeanName);
    scanner.setResourceLoader(this.applicationContext);
    scanner.setBeanNameGenerator(this.nameGenerator);
    scanner.setMapperFactoryBeanClass(this.mapperFactoryBeanClass);
    if (StringUtils.hasText(this.lazyInitialization)) {
        scanner.setLazyInitialization(Boolean.valueOf(this.lazyInitialization));
    }

    if (StringUtils.hasText(this.defaultScope)) {
        scanner.setDefaultScope(this.defaultScope);
    }

  	//添加过滤器，这里会配置为所有的接口都能被扫描（因此即使你不添加@Mapper注解都能够被扫描并加载）
    scanner.registerFilters();
  	//开始扫描
    scanner.scan(StringUtils.tokenizeToStringArray(this.basePackage, ",; \t\n"));
}
```

扫描`scan()`会调用`doScan()`方法，来看看（这是`ClassPathMapperScanner`中的扫描方法）

```java
public Set<BeanDefinitionHolder> doScan(String... basePackages) {
    Set<BeanDefinitionHolder> beanDefinitions = super.doScan(basePackages);
  	//首先从包中扫描所有的Bean定义
    if (beanDefinitions.isEmpty()) {
        LOGGER.warn(() -> {
            return "No MyBatis mapper was found in '" + Arrays.toString(basePackages) + "' package. Please check your configuration.";
        });
    } else {
      	//处理所有的Bean定义，实际上就是生成对应Mapper的代理对象，并注册到容器中
        this.processBeanDefinitions(beanDefinitions);
    }

  	//最后返回所有的Bean定义集合
    return beanDefinitions;
}
```

通过断点发现，最后处理得到的Bean定义发现此Bean是一个MapperFactoryBean，它不同于普通的Bean，FactoryBean相当于为普通的Bean添加了一层外壳，它并不是依靠Spring直接通过反射创建，而是使用接口中的方法：

```java
public interface FactoryBean<T> {
    String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

    @Nullable
    T getObject() throws Exception;

    @Nullable
    Class<?> getObjectType();

    default boolean isSingleton() {
        return true;
    }
}
```

通过`getObject()`方法，就可以获取到Bean的实例了。

*区分FactoryBean和BeanFactory的概念：*

* BeanFactory是个Factory，也就是 IOC 容器或对象工厂，所有的 Bean 都是由 BeanFactory( 也就是 IOC 容器 ) 来进行管理。
* FactoryBean是一个能生产或者修饰生成对象的工厂Bean(本质上也是一个Bean)，可以在BeanFactory（IOC容器）中被管理，所以它并不是一个简单的Bean。当使用容器中factory bean的时候，该容器不会返回factory bean本身，而是返回其生成的对象。要想获取FactoryBean的实现类本身，得在getBean(String BeanName)中的BeanName之前加上&,写成getBean(String &BeanName)。

可以自己编写一个实现：

```java
@Component("test")
public class TestFb implements FactoryBean<Student> {
    @Override
    public Student getObject() throws Exception {
        System.out.println("获取了学生");
        return new Student();
    }

    @Override
    public Class<?> getObjectType() {
        return Student.class;
    }
}
```

```java
public static void main(String[] args) {
    log.info("项目正在启动...");
    ApplicationContext context = new AnnotationConfigApplicationContext(TestConfiguration.class);
    System.out.println(context.getBean("&test"));   //得到FactoryBean本身（得加个&搞得像C语言指针一样）
    System.out.println(context.getBean("test"));   //得到FactoryBean调用getObject()之后的结果
}
```

实际上我们的Mapper最终就以FactoryBean的形式，被注册到容器中进行加载了：

```java
public T getObject() throws Exception {
    return this.getSqlSession().getMapper(this.mapperInterface);
}
```

这样，整个Mybatis的`@MapperScan`的原理就全部解释完毕了。

## SpringMVC原理——分析DispatcherServlet源码

*注意*： 大型框架的底层一般都是层层套娃，因为这样写起来层次会更加清晰。


**DispatcherServlet初始化过程**

`DispatcherServlet`的最顶层是`HttpServletBean`，直接继承了`HttpServlet`

首先来看一下，它在初始化方法中做了什么：

```java
public final void init() throws ServletException {
  	//读取配置参数，并进行配置
    PropertyValues pvs = new HttpServletBean.ServletConfigPropertyValues(this.getServletConfig(), this.requiredProperties);
    if (!pvs.isEmpty()) {
        try {
            BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
            ResourceLoader resourceLoader = new ServletContextResourceLoader(this.getServletContext());
            bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, this.getEnvironment()));
            this.initBeanWrapper(bw);
            bw.setPropertyValues(pvs, true);
        } catch (BeansException var4) {
            if (this.logger.isErrorEnabled()) {
                this.logger.error("Failed to set bean properties on servlet '" + this.getServletName() + "'", var4);
            }

            throw var4;
        }
    }
		//此初始化阶段由子类实现，
    this.initServletBean();
}
```

接着来看`initServletBean()`方法是如何实现的，它是在子类`FrameworkServlet`中定义的：

```java
protected final void initServletBean() throws ServletException {
    this.getServletContext().log("Initializing Spring " + this.getClass().getSimpleName() + " '" + this.getServletName() + "'");
    if (this.logger.isInfoEnabled()) {
        this.logger.info("Initializing Servlet '" + this.getServletName() + "'");
    }

    long startTime = System.currentTimeMillis();

    try {
      	//注意：SpringMVC有两个容器，一个是Web容器一个是根容器
      	//Web容器只负责Controller等表现层内容
      	//根容器就是Spring容器，它负责Service、Dao等，并且它是Web容器的父容器。
      	//初始化WebApplicationContext，这个阶段会为根容器和Web容器进行父子关系建立
        this.webApplicationContext = this.initWebApplicationContext();
        this.initFrameworkServlet();
    } catch (RuntimeException | ServletException var4) {
      //...以下内容全是打印日志
}
```

![img](https://images2018.cnblogs.com/blog/1290804/201712/1290804-20171209164442730-1374080285.png)

看看`initWebApplicationContext`是如何进行初始化的：

```java
protected WebApplicationContext initWebApplicationContext() {
  	//这里获取的是根容器，一般用于配置Service、数据源等
    WebApplicationContext rootContext = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext());
    WebApplicationContext wac = null;//这个是web容器
    if (this.webApplicationContext != null) {
      	//如果webApplicationContext在之前已经存在，则直接给到wac
        wac = this.webApplicationContext;
        if (wac instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext)wac;
            if (!cwac.isActive()) {
                if (cwac.getParent() == null) {
                  	//设定根容器为Web容器的父容器
                    cwac.setParent(rootContext);
                }

                this.configureAndRefreshWebApplicationContext(cwac);
            }
        }
    }

    if (wac == null) {
      	//如果webApplicationContext是空，那么就从ServletContext找一下有没有初始化上下文
        wac = this.findWebApplicationContext();
    }

    if (wac == null) {
      	//如果还是找不到，直接创个新的，并直接将根容器作为父容器
        wac = this.createWebApplicationContext(rootContext);
    }

    if (!this.refreshEventReceived) {
        synchronized(this.onRefreshMonitor) {
          	//此方法由DispatcherServlet实现
            this.onRefresh(wac);
        }
    }

    if (this.publishContext) {
        String attrName = this.getServletContextAttributeName();
      	//把Web容器丢进ServletContext
        this.getServletContext().setAttribute(attrName, wac);
    }

    return wac;
}
```

接着来看DispatcherServlet中实现的`onRefresh()`方法：

```java
@Override
protected void onRefresh(ApplicationContext context) {
    initStrategies(context);
}
    
protected void initStrategies(ApplicationContext context) {
  	//初始化各种解析器
    initMultipartResolver(context);
    initLocaleResolver(context);
    initThemeResolver(context);
  	//在容器中查找所有的HandlerMapping，放入集合中
  	//HandlerMapping保存了所有的请求映射信息（Controller中定义的），它可以根据请求找到处理器Handler，但并不是简单的返回处理器，而是将处理器和拦截器封装，形成一个处理器执行链（类似于之前的Filter）
    initHandlerMappings(context);
  	//在容器中查找所有的HandlerAdapter，它用于处理请求并返回ModelAndView对象（无论什么类型，最终都会被封装成它）
  	//默认有三种实现HttpRequestHandlerAdapter，SimpleControllerHandlerAdapter和AnnotationMethodHandlerAdapter
  	//当HandlerMapping找到处理请求的Controller之后，会选择一个合适的HandlerAdapter处理请求
  	//比如之前使用的是注解方式配置Controller（那么就会用AnnotationMethodHandlerAdapter处理请求），现在有一个请求携带了一个参数，那么HandlerAdapter会对请求的数据进行解析，并传入方法作为实参，最后根据方法的返回值将其封装为ModelAndView对象
    initHandlerAdapters(context);
  	//其他的内容
    initHandlerExceptionResolvers(context);
    initRequestToViewNameTranslator(context);
    initViewResolvers(context);
    initFlashMapManager(context);
}
```

**DispatcherServlet是如何进行调度的**

首先请求肯定会经过`HttpServlet`，然后其交给对应的`doGet`、`doPost`等方法进行处理。

在`FrameworkServlet`中，这些方法都被重写，并且都使用`processRequest`来进行处理：

```java
protected final void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    this.processRequest(request, response);
}

protected final void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    this.processRequest(request, response);
}
```

看看`processRequest`做了什么：

```java
protected final void processRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
  	//前期准备工作
    long startTime = System.currentTimeMillis();
    Throwable failureCause = null;
    LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
    LocaleContext localeContext = this.buildLocaleContext(request);
    RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
    ServletRequestAttributes requestAttributes = this.buildRequestAttributes(request, response, previousAttributes);
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new FrameworkServlet.RequestBindingInterceptor());
    this.initContextHolders(request, localeContext, requestAttributes);

    try {
      	//重点在这里，这里进行了Service的执行，不过是在DispatcherServlet中定义的
        this.doService(request, response);
    } catch (IOException | ServletException var16) {
        //...
}
```

看看`DispatcherServlet`中的doService()是如何实现的：

```java
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
   //...
    try {
      	//重点在这里，这才是整个处理过程中最核心的部分
        this.doDispatch(request, response);
    } finally {
        //...
}
```

最核心的部分是`doDispatch()`：

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
        try {
            ModelAndView mv = null;
            Object dispatchException = null;

            try {
                processedRequest = this.checkMultipart(request);
                multipartRequestParsed = processedRequest != request;
              	//在HandlerMapping集合中寻找可以处理当前请求的HandlerMapping
                mappedHandler = this.getHandler(processedRequest);
                if (mappedHandler == null) {
                    this.noHandlerFound(processedRequest, response);
                  	//找不到HandlerMapping则无法进行处理
                    return;
                }

              	//根据HandlerMapping提供的信息，找到可以处理的HandlerAdapter
                HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());
                String method = request.getMethod();
                boolean isGet = HttpMethod.GET.matches(method);
                if (isGet || HttpMethod.HEAD.matches(method)) {
                    long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                    if ((new ServletWebRequest(request, response)).checkNotModified(lastModified) && isGet) {
                        return;
                    }
                }

              	//执行所有拦截器的preHandle()方法
                if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                    return;
                }

              	//使用HandlerAdapter进行处理（我们编写的请求映射方法在这个位置才真正地执行了）
              	//HandlerAdapter会帮助我们将请求的数据进行处理，再来调用我们编写的请求映射方法
              	//最后HandlerAdapter会将结果封装为ModelAndView返回给mv
                mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
                if (asyncManager.isConcurrentHandlingStarted()) {
                    return;
                }

                this.applyDefaultViewName(processedRequest, mv);
              	//执行所有拦截器的postHandle()方法
                mappedHandler.applyPostHandle(processedRequest, response, mv);
            } catch (Exception var20) {
                dispatchException = var20;
            } catch (Throwable var21) {
                dispatchException = new NestedServletException("Handler dispatch failed", var21);
            }

          	//最后处理结果，对视图进行渲染等，如果抛出异常会出现错误页面
            this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);
        } catch (Exception var22) {
            this.triggerAfterCompletion(processedRequest, response, mappedHandler, var22);
        } catch (Throwable var23) {
            this.triggerAfterCompletion(processedRequest, response, mappedHandler, new NestedServletException("Handler processing failed", var23));
        }

    } finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            if (mappedHandler != null) {
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            }
        } else if (multipartRequestParsed) {
            this.cleanupMultipart(processedRequest);
        }

    }
}
```

以上源码分析得出最终的流程图：

![img](https://img2018.cnblogs.com/blog/1338162/201901/1338162-20190113192808388-307235311.png)

1. 浏览器发出请求给到中央调度器DispatcherServlet
2. 中央调度器把这个请求给到处理器映射器HandleMapping，如果找到存在controller里面有对应的请求映射（`@RequestMapping`URL与请求一致），则生成处理器执行链（将处理器和拦截器封装），给回中央调度器
3. 中央调度器拿到处理器执行链后，将它给到对应的处理器适配器HandleAdaptor
4. 处理器适配器将请求参数，请求的返回值等做一些处理（比如将请求参数作为实参传入到Controller的请求方法里），给到处理器Controller
5. 处理器方法返回的结果包装成ModelAndView对象，给到中央调度器
6. 如果ModelAndView对象非null，那么中央调度器将这个对象给到视图解析器ViewResolver进行解析得到页面View
7. 视图解析器把页面View给到中央调度器，最后中央调度器把这个页面响应给浏览器

## SpringSecurity原理

SpringSecurity本质上是依靠N个Filter实现的，也就是一个完整的过滤链（注意这里是过滤器，不是拦截器）

看看`AbstractSecurityWebApplicationInitializer`配置了什么

```java
//此方法会在启动时被调用
public final void onStartup(ServletContext servletContext) {
    this.beforeSpringSecurityFilterChain(servletContext);
    if (this.configurationClasses != null) {
        AnnotationConfigWebApplicationContext rootAppContext = new AnnotationConfigWebApplicationContext();
        rootAppContext.register(this.configurationClasses);
        servletContext.addListener(new ContextLoaderListener(rootAppContext));
    }

    if (this.enableHttpSessionEventPublisher()) {
        servletContext.addListener("org.springframework.security.web.session.HttpSessionEventPublisher");
    }

    servletContext.setSessionTrackingModes(this.getSessionTrackingModes());
  	//重点在这里，这里插入了关键的FilterChain
    this.insertSpringSecurityFilterChain(servletContext);
    this.afterSpringSecurityFilterChain(servletContext);
}
```

```java
private void insertSpringSecurityFilterChain(ServletContext servletContext) {
    String filterName = "springSecurityFilterChain";
  	//创建了一个DelegatingFilterProxy对象，它本质上也是一个Filter
    DelegatingFilterProxy springSecurityFilterChain = new DelegatingFilterProxy(filterName);
    String contextAttribute = this.getWebApplicationContextAttribute();
    if (contextAttribute != null) {
        springSecurityFilterChain.setContextAttribute(contextAttribute);
    }
		//通过ServletContext注册DelegatingFilterProxy这个Filter
    this.registerFilter(servletContext, true, filterName, springSecurityFilterChain);
}
```

接着看看`DelegatingFilterProxy`在做什么：

```java
//这个是初始化方法，它由GenericFilterBean（父类）定义，在afterPropertiesSet方法中被调用
protected void initFilterBean() throws ServletException {
    synchronized(this.delegateMonitor) {
        if (this.delegate == null) {
            if (this.targetBeanName == null) {
                this.targetBeanName = this.getFilterName();
            }

            WebApplicationContext wac = this.findWebApplicationContext();
            if (wac != null) {
              	//耐心点，套娃很正常
                this.delegate = this.initDelegate(wac);
            }
        }

    }
}
```

```java
protected Filter initDelegate(WebApplicationContext wac) throws ServletException {
    String targetBeanName = this.getTargetBeanName();
    Assert.state(targetBeanName != null, "No target bean name set");
  	//这里通过WebApplicationContext获取了一个Bean
    Filter delegate = (Filter)wac.getBean(targetBeanName, Filter.class);
    if (this.isTargetFilterLifecycle()) {
        delegate.init(this.getFilterConfig());
    }

  	//返回Filter
    return delegate;
}
```

通过断点调试，发现这里返回的对象是一个`FilterChainProxy`类型的，并且调用了它的初始化方法，但是FilterChainProxy类中并没有重写`init`方法或是`initFilterBean`方法。

倒回去看，当Filter返回之后，`DelegatingFilterProxy`的一个成员变量`delegate`被赋值为得到的Filter，也就是FilterChainProxy对象

接着来看看，`DelegatingFilterProxy`是如何执行doFilter方法的。

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws ServletException, IOException {
    Filter delegateToUse = this.delegate;
    if (delegateToUse == null) {
        //非正常情况，这里省略...
    }
		//这里才是真正的调用，别忘了delegateToUse就是初始化的FilterChainProxy对象
    this.invokeDelegate(delegateToUse, request, response, filterChain);
}
```

```java
protected void invokeDelegate(Filter delegate, ServletRequest request, ServletResponse response, FilterChain filterChain) throws ServletException, IOException {
  //最后实际上调用的是FilterChainProxy的doFilter方法
    delegate.doFilter(request, response, filterChain);
}
```

所以接着来看，`FilterChainProxy`的doFilter方法又在干什么：

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    boolean clearContext = request.getAttribute(FILTER_APPLIED) == null;
    if (!clearContext) {
      	//真正的过滤在这里执行
        this.doFilterInternal(request, response, chain);
    } else {
        //...
    }
}
```

```java
private void doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    FirewalledRequest firewallRequest = this.firewall.getFirewalledRequest((HttpServletRequest)request);
    HttpServletResponse firewallResponse = this.firewall.getFirewalledResponse((HttpServletResponse)response);
  	//这里获取了一个Filter列表，实际上SpringSecurity就是由N个过滤器实现的，这里获取的都是SpringSecurity提供的过滤器
  	//但是经过之前的分析，实际上真正注册的Filter只有DelegatingFilterProxy
  	//而这里的Filter列表中的所有Filter并没有被注册，而是在这里进行内部调用
    List<Filter> filters = this.getFilters((HttpServletRequest)firewallRequest);
  	//只要Filter列表不是空，就依次执行内置的Filter
    if (filters != null && filters.size() != 0) {
        if (logger.isDebugEnabled()) {
            logger.debug(LogMessage.of(() -> {
                return "Securing " + requestLine(firewallRequest);
            }));
        }
				//这里创建一个虚拟的过滤链，过滤流程是由SpringSecurity自己实现的
        FilterChainProxy.VirtualFilterChain virtualFilterChain = new FilterChainProxy.VirtualFilterChain(firewallRequest, chain, filters);
      	//调用虚拟过滤链的doFilter
        virtualFilterChain.doFilter(firewallRequest, firewallResponse);
    } else {
        if (logger.isTraceEnabled()) {
            logger.trace(LogMessage.of(() -> {
                return "No security for " + requestLine(firewallRequest);
            }));
        }

        firewallRequest.reset();
        chain.doFilter(firewallRequest, firewallResponse);
    }
}
```

接着来看一下这个虚拟过滤链的doFilter是怎么处理的：

```java
//看似没有任何循环，实际上就是一个循环，是一个递归调用
public void doFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException {
  	//判断是否已经通过全部的内置过滤器，定位是否等于当前大小
    if (this.currentPosition == this.size) {
        if (FilterChainProxy.logger.isDebugEnabled()) {
            FilterChainProxy.logger.debug(LogMessage.of(() -> {
                return "Secured " + FilterChainProxy.requestLine(this.firewalledRequest);
            }));
        }

        this.firewalledRequest.reset();
      	//所有的内置过滤器已经完成，按照正常流程走DelegatingFilterProxy的下一个Filter
      	//也就是说这里之后就与DelegatingFilterProxy没有任何关系了，该走其他过滤器就走其他地方配置的过滤器，SpringSecurity的过滤操作已经结束
        this.originalChain.doFilter(request, response);
    } else {
      	//定位自增
        ++this.currentPosition;
      	//获取当前定位的Filter
        Filter nextFilter = (Filter)this.additionalFilters.get(this.currentPosition - 1);
        if (FilterChainProxy.logger.isTraceEnabled()) {
            FilterChainProxy.logger.trace(LogMessage.format("Invoking %s (%d/%d)", nextFilter.getClass().getSimpleName(), this.currentPosition, this.size));
        }
				//执行内部过滤器的doFilter方法，传入当前对象本身作为Filter，执行如果成功，那么一定会再次调用当前对象的doFilter方法
      	//可能最不理解的就是这里，执行的难道不是内部其他Filter的doFilter方法吗，怎么会让当前对象的doFilter方法递归调用呢？
      	//了解了其中一个内部过滤器就明白了
        nextFilter.doFilter(request, response, this);
    }
}
```

现在差不多已经了解了整个SpringSecurity的实现机制了，那么来看几个内部的过滤器分别在做什么。

比如用于处理登陆的过滤器`UsernamePasswordAuthenticationFilter`，它继承自`AbstractAuthenticationProcessingFilter`：

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    this.doFilter((HttpServletRequest)request, (HttpServletResponse)response, chain);
}

private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
  	//如果不是登陆请求，那么根本不会理这个请求
    if (!this.requiresAuthentication(request, response)) {
      	//直接调用传入的FilterChain的doFilter方法
      	//而这里传入的正好是VirtualFilterChain对象
      	//这下就明白为什么之前说虚拟过滤链的doFilter是递归了
        chain.doFilter(request, response);
    } else {
      	//如果是登陆请求，那么会执行登陆请求的相关逻辑，注意执行过程中出现任何问题都会抛出异常
      	//比如用户名和密码错误，会得到一个BadCredentialsException
        try {
          	//进行认证
            Authentication authenticationResult = this.attemptAuthentication(request, response);
            if (authenticationResult == null) {
                return;
            }

            this.sessionStrategy.onAuthentication(authenticationResult, request, response);
            if (this.continueChainBeforeSuccessfulAuthentication) {
                chain.doFilter(request, response);
            }

          	//如果一路绿灯，没有报错，那么验证成功，执行successfulAuthentication
            this.successfulAuthentication(request, response, chain, authenticationResult);
        } catch (InternalAuthenticationServiceException var5) {
            this.logger.error("An internal error occurred while trying to authenticate the user.", var5);
          	//验证失败，会执行unsuccessfulAuthentication
            this.unsuccessfulAuthentication(request, response, var5);
        } catch (AuthenticationException var6) {
            this.unsuccessfulAuthentication(request, response, var6);
        }

    }
}
```

来看看successfulAuthentication和unsuccessfulAuthentication分别做了什么：

```java
protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
  	//向SecurityContextHolder添加认证信息，我们可以通过SecurityContextHolder对象获取当前登陆的用户
    SecurityContextHolder.getContext().setAuthentication(authResult);
    if (this.logger.isDebugEnabled()) {
        this.logger.debug(LogMessage.format("Set SecurityContextHolder to %s", authResult));
    }

  	//记住我实现
    this.rememberMeServices.loginSuccess(request, response, authResult);
    if (this.eventPublisher != null) {
        this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));
    }
		
  	//调用默认的或是我们自己定义的AuthenticationSuccessHandler的onAuthenticationSuccess方法
  	//这个根据我们配置文件决定
  	//到这里其实页面就已经直接跳转了
    this.successHandler.onAuthenticationSuccess(request, response, authResult);
}

protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) throws IOException, ServletException {
  	//登陆失败会直接清理掉SecurityContextHolder中的认证信息
    SecurityContextHolder.clearContext();
    this.logger.trace("Failed to process authentication request", failed);
    this.logger.trace("Cleared SecurityContextHolder");
    this.logger.trace("Handling authentication failure");
  	//登陆失败的记住我处理
    this.rememberMeServices.loginFail(request, response);
  	//同上，调用默认或是我们自己定义的AuthenticationFailureHandler
    this.failureHandler.onAuthenticationFailure(request, response, failed);
}
```

了解了整个用户验证实现流程，其实其它的过滤器是如何实现的也就很容易联想到了，SpringSecurity的过滤器从某种意义上来说，更像是一个处理业务的Servlet，它做的事情不像是拦截，更像是完成自己对应的职责，只不过是使用了过滤器机制进行实现罢了。

SecurityContextPersistenceFilter也是内置的Filter，可以尝试阅读一下其源码，了解整个SecurityContextHolder的运作原理。大致流程：

当过滤器链执行到SecurityContextPersistenceFilter时，它会从HttpSession中把SecurityContext对象取出来（是存在Session中的，跟随会话的消失而消失），然后放入SecurityContextHolder对象中。请求结束后，再把SecurityContext存入HttpSession中，并清除SecurityContextHolder内的SecurityContext对象。

## 探究SpringBoot实现原理

### 启动原理

SpringBoot项目启动之后，做了什么事情

SpringApplication中的静态`run`方法：

```java
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
    return run(new Class[]{primarySource}, args);
}
```

套娃如下：

这里直接new了一个新的SpringApplication对象，传入我们的主类作为构造方法参数，并调用了非static的`run`方法，

```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    return (new SpringApplication(primarySources)).run(args);
}
```

先来看看SpringApplication构造方法里面做了什么事情：

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    ...
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));
  	//这里是关键，这里会判断当前SpringBoot应用程序是否为Web项目，并返回当前的项目类型
  	//deduceFromClasspath是根据类路径下判断是否包含SpringBootWeb依赖，如果不包含就是NONE类型，包含就是SERVLET类型
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    this.bootstrapRegistryInitializers = new ArrayList(this.getSpringFactoriesInstances(BootstrapRegistryInitializer.class));
  	//创建所有ApplicationContextInitializer实现类的对象
    this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
    this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
    this.mainApplicationClass = this.deduceMainApplicationClass();
}
```

关键就在这里了，它是如何知道哪些类是ApplicationContextInitializer的实现类的呢？

这里就要提到spring.factories，它是 Spring 仿造Java SPI实现的一种类加载机制。它在 META-INF/spring.factories 文件中配置接口的实现类名称，然后在程序中读取这些配置文件并实例化。这种自定义的SPI机制是 Spring Boot Starter 实现的基础。

SPI的常见例子：

- 数据库驱动加载接口实现类的加载：JDBC加载不同类型数据库的驱动
- 日志门面接口实现类加载：SLF4J加载不同提供商的日志实现类

说白了就是人家定义接口，实现可能有很多种，但是核心只提供接口，需要我们按需选择对应的实现，这种方式是高度解耦的。

看看`getSpringFactoriesInstances`方法做了什么：

其中`SpringFactoriesLoader.loadFactoryNames`正是读取配置的核心部分

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
  	//获取当前的类加载器
    ClassLoader classLoader = this.getClassLoader();
  	//获取所有依赖中 META-INF/spring.factories 中配置的对应接口类的实现类列表
    Set<String> names = new LinkedHashSet(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
  	//根据上方列表，依次创建实例对象  
  List<T> instances = this.createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
  	//根据对应类上的Order接口或是注解进行排序
    AnnotationAwareOrderComparator.sort(instances);
  	//返回实例
    return instances;
}
```

接着来看run方法里面做了什么事情。

```java
public ConfigurableApplicationContext run(String... args) {
    long startTime = System.nanoTime();
    DefaultBootstrapContext bootstrapContext = this.createBootstrapContext();
    ConfigurableApplicationContext context = null;
    this.configureHeadlessProperty();
  	//获取所有的SpringApplicationRunListener，并通知启动事件，默认只有一个实现类EventPublishingRunListener
  	//EventPublishingRunListener会将初始化各个阶段的事件转发给所有监听器
    SpringApplicationRunListeners listeners = this.getRunListeners(args);
    listeners.starting(bootstrapContext, this.mainApplicationClass);

    try {
      	//环境配置
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);
        this.configureIgnoreBeanInfo(environment);
      	//打印Banner
        Banner printedBanner = this.printBanner(environment);
      	//创建ApplicationContext，注意这里会根据是否为Web容器使用不同的ApplicationContext实现类
        context = this.createApplicationContext();
        context.setApplicationStartup(this.applicationStartup);
      	//初始化ApplicationContext
        this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
      	//执行ApplicationContext的refresh方法
        this.refreshContext(context);
        this.afterRefresh(context, applicationArguments);
        Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
        if (this.logStartupInfo) {
            (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), timeTakenToStartup);
        }
        ....
}
```

实际上SpringBoot就是Spring的一层壳罢了，离不开最关键的ApplicationContext，也就是说，在启动后会自动配置一个ApplicationContext，只不过是进行了大量的扩展。

来看看ApplicationContext是怎么来的，打开`createApplicationContext`方法：

```java
protected ConfigurableApplicationContext createApplicationContext() {
    return this.applicationContextFactory.create(this.webApplicationType);
}
```

在构造方法中`applicationContextFactory`直接使用的是DEFAULT：

```java
this.applicationContextFactory = ApplicationContextFactory.DEFAULT;
```

```java
ApplicationContextFactory DEFAULT = (webApplicationType) -> {
    try {
        switch(webApplicationType) {
        case SERVLET:
            return new AnnotationConfigServletWebServerApplicationContext();
        case REACTIVE:
            return new AnnotationConfigReactiveWebServerApplicationContext();
        default:
            return new AnnotationConfigApplicationContext();
        }
    } catch (Exception var2) {
        throw new IllegalStateException("Unable create a default ApplicationContext instance, you may need a custom ApplicationContextFactory", var2);
    }
};

ConfigurableApplicationContext create(WebApplicationType webApplicationType);
```

DEFAULT是直接编写的一个匿名内部类，其实已经很明确了，正是根据`webApplicationType`类型进行判断。如果是SERVLET，那么就返回专用于Web环境的AnnotationConfigServletWebServerApplicationContext对象（SpringBoot中新增的），否则返回普通的AnnotationConfigApplicationContext对象。也就是到这里为止，Spring的容器就基本已经确定了。

AnnotationConfigApplicationContext是Spring框架提供的类，从这里开始就是Spring的底层源码了。

AnnotationConfigApplicationContext对象在创建过程中会创建`AnnotatedBeanDefinitionReader`，它是用于通过注解解析Bean定义的工具类：

```java
public AnnotationConfigApplicationContext() {
    StartupStep createAnnotatedBeanDefReader = this.getApplicationStartup().start("spring.context.annotated-bean-reader.create");
    this.reader = new AnnotatedBeanDefinitionReader(this);
    createAnnotatedBeanDefReader.end();
    this.scanner = new ClassPathBeanDefinitionScanner(this);
}
```

其构造方法：

```java
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
    ...
    //这里会注册很多的后置处理器
    AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}
```

```java
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(BeanDefinitionRegistry registry, @Nullable Object source) {
    DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
    ....
    Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet(8);
    RootBeanDefinition def;
    if (!registry.containsBeanDefinition("org.springframework.context.annotation.internalConfigurationAnnotationProcessor")) {
      	//注册了ConfigurationClassPostProcessor用于处理@Configuration、@Import等注解
      	//注意这里是关键，之后Selector还要讲到它
      	//它是继承自BeanDefinitionRegistryPostProcessor，所以它的执行时间在Bean定义加载完成后，Bean初始化之前
        def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalConfigurationAnnotationProcessor"));
    }

    if (!registry.containsBeanDefinition("org.springframework.context.annotation.internalAutowiredAnnotationProcessor")) {
      	//AutowiredAnnotationBeanPostProcessor用于处理@Value等注解自动注入
        def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalAutowiredAnnotationProcessor"));
    }
  
  	...
```

回到SpringBoot，最后来看，`prepareContext`方法中又做了什么事情：

```java
private void prepareContext(DefaultBootstrapContext bootstrapContext, ConfigurableApplicationContext context, ConfigurableEnvironment environment, SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
  	//环境配置
    context.setEnvironment(environment);
    this.postProcessApplicationContext(context);
    this.applyInitializers(context);
    listeners.contextPrepared(context);
    bootstrapContext.close(context);
    if (this.logStartupInfo) {
        this.logStartupInfo(context.getParent() == null);
        this.logStartupProfileInfo(context);
    }

  	//将Banner注册为Bean
    ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
    beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
    if (printedBanner != null) {
        beanFactory.registerSingleton("springBootBanner", printedBanner);
    }

    if (beanFactory instanceof AbstractAutowireCapableBeanFactory) {
        ((AbstractAutowireCapableBeanFactory)beanFactory).setAllowCircularReferences(this.allowCircularReferences);
        if (beanFactory instanceof DefaultListableBeanFactory) {
            ((DefaultListableBeanFactory)beanFactory).setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
        }
    }

    if (this.lazyInitialization) {
        context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
    }

  	//这里会获取我们一开始传入的项目主类
    Set<Object> sources = this.getAllSources();
    Assert.notEmpty(sources, "Sources must not be empty");
  	//这里会将我们的主类直接注册为Bean，这样就可以通过注解加载了
    this.load(context, sources.toArray(new Object[0]));
    listeners.contextLoaded(context);
}
```

因此，在`prepareContext`执行完成之后，我们的主类成功完成Bean注册。

### 自动配置原理

既然主类已经在初始阶段注册为Bean，那么在加载时，就会根据注解定义，进行更多的额外操作。

所以来看看主类上的`@SpringBootApplication`注解做了什么事情。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
```

`@SpringBootApplication`上添加了`@ComponentScan`注解，但是这里并没有配置具体扫描的包，因此它会自动将声明此接口的类所在的包作为basePackage。也就是说，当添加`@SpringBootApplication`之后也就等于直接开启了自动扫描，但是一定注意不能在主类之外的包进行Bean定义，否则无法扫描到，需要手动配置。

接着来看第二个注解`@EnableAutoConfiguration`，它就是自动配置的核心。
定义如下：

和以前一样，通过`@Import`这种方式来将一些外部的Bean加载到容器中。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
```

看看AutoConfigurationImportSelector做了什么事情：

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
		...
}
```

我们看到它实现了很多接口，包括大量的Aware接口，实际上就是为了感知某些必要的对象，并将其存到当前类中。

其中最核心的是`DeferredImportSelector`接口，它是`ImportSelector`的子类，定义了`selectImports`方法，用于返回需要加载的类名称。在Spring加载ImportSelector类型的Bean时，会调用此方法来获取更多需要加载的类，并将这些类一并注册为Bean：

```java
public interface ImportSelector {
    String[] selectImports(AnnotationMetadata importingClassMetadata);

    @Nullable
    default Predicate<String> getExclusionFilter() {
        return null;
    }
}
```

目前为止了解了两种使用`@Import`有特殊机制的接口：ImportSelector（这里用到的）和ImportBeanDefinitionRegistrar（之前Mybatis-spring源码有讲）当然还有普通的`@Configuration`配置类。

阅读一下`ConfigurationClassPostProcessor`的源码，看看它到底是如何处理`@Import`的：

```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    List<BeanDefinitionHolder> configCandidates = new ArrayList();
  	//注意这个阶段仅仅是已经完成扫描了所有的Bean，得到了所有的BeanDefinition，但是还没有进行任何区分
  	//candidate是候选者的意思，一会会将标记了@Configuration的类作为ConfigurationClass加入到configCandidates中
    String[] candidateNames = registry.getBeanDefinitionNames();
    String[] var4 = candidateNames;
    int var5 = candidateNames.length;

    for(int var6 = 0; var6 < var5; ++var6) {
        String beanName = var4[var6];
        BeanDefinition beanDef = registry.getBeanDefinition(beanName);
        if (beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE) != null) {
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Bean definition has already been processed as a configuration class: " + beanDef);
            }
        } else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {   //判断是否添加了@Configuration注解
            configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
        }
    }

    if (!configCandidates.isEmpty()) {
        //...省略

      	//这里创建了一个ConfigurationClassParser用于解析配置类
        ConfigurationClassParser parser = new ConfigurationClassParser(this.metadataReaderFactory, this.problemReporter, this.environment, this.resourceLoader, this.componentScanBeanNameGenerator, registry);
      	//所有配置类的BeanDefinitionHolder列表
        Set<BeanDefinitionHolder> candidates = new LinkedHashSet(configCandidates);
      	//已经解析完成的类
        HashSet alreadyParsed = new HashSet(configCandidates.size());

        do {
            //这里省略，直到所有的配置类全部解析完成
          	//注意在循环过程中可能会由于@Import新增更多的待解析配置类，一律丢进candidates集合中
        } while(!candidates.isEmpty());

        ...

    }
}
```

接着来看，`ConfigurationClassParser`是如何进行解析的：

```java
protected void processConfigurationClass(ConfigurationClass configClass, Predicate<String> filter) throws IOException {
  	//@Conditional相关注解处理
  	//后面会讲
    if (!this.conditionEvaluator.shouldSkip(configClass.getMetadata(), ConfigurationPhase.PARSE_CONFIGURATION)) {
        ...
        }

        ConfigurationClassParser.SourceClass sourceClass = this.asSourceClass(configClass, filter);

        do {
          	//核心
            sourceClass = this.doProcessConfigurationClass(configClass, sourceClass, filter);
        } while(sourceClass != null);

        this.configurationClasses.put(configClass, configClass);
    }
}
```

最后再来看最核心的`doProcessConfigurationClass`方法：

```java
protected final SourceClass doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass)
    ...

    processImports(configClass, sourceClass, getImports(sourceClass), true);    // 处理Import注解

		...

    return null;
}
```

```java
private void processImports(ConfigurationClass configClass, ConfigurationClassParser.SourceClass currentSourceClass, Collection<ConfigurationClassParser.SourceClass> importCandidates, Predicate<String> exclusionFilter, boolean checkForCircularImports) {
    if (!importCandidates.isEmpty()) {
        if (checkForCircularImports && this.isChainedImportOnStack(configClass)) {
            this.problemReporter.error(new ConfigurationClassParser.CircularImportProblem(configClass, this.importStack));
        } else {
            this.importStack.push(configClass);

            try {
                Iterator var6 = importCandidates.iterator();

                while(var6.hasNext()) {
                    ConfigurationClassParser.SourceClass candidate = (ConfigurationClassParser.SourceClass)var6.next();
                    Class candidateClass;
                  	//如果是ImportSelector类型，继续进行运行
                    if (candidate.isAssignable(ImportSelector.class)) {
                        candidateClass = candidate.loadClass();
                        ImportSelector selector = (ImportSelector)ParserStrategyUtils.instantiateClass(candidateClass, ImportSelector.class, this.environment, this.resourceLoader, this.registry);
                        Predicate<String> selectorFilter = selector.getExclusionFilter();
                        if (selectorFilter != null) {
                            exclusionFilter = exclusionFilter.or(selectorFilter);
                        }
									//如果是DeferredImportSelector的实现类，那么会走deferredImportSelectorHandler的handle方法
                        if (selector instanceof DeferredImportSelector) {
                            this.deferredImportSelectorHandler.handle(configClass, (DeferredImportSelector)selector);
                          //否则就按照正常的ImportSelector类型进行加载
                        } else {
                          	//调用selectImports方法获取所有需要加载的类
                            String[] importClassNames = selector.selectImports(currentSourceClass.getMetadata());
                            Collection<ConfigurationClassParser.SourceClass> importSourceClasses = this.asSourceClasses(importClassNames, exclusionFilter);
                          	//递归处理，直到没有
                            this.processImports(configClass, currentSourceClass, importSourceClasses, exclusionFilter, false);
                        }
                      //判断是否为ImportBeanDefinitionRegistrar类型
                    } else if (candidate.isAssignable(ImportBeanDefinitionRegistrar.class)) {
                        candidateClass = candidate.loadClass();
                        ImportBeanDefinitionRegistrar registrar = (ImportBeanDefinitionRegistrar)ParserStrategyUtils.instantiateClass(candidateClass, ImportBeanDefinitionRegistrar.class, this.environment, this.resourceLoader, this.registry);
                      	//往configClass丢ImportBeanDefinitionRegistrar信息进去，之后再处理
                        configClass.addImportBeanDefinitionRegistrar(registrar, currentSourceClass.getMetadata());
                      //否则按普通的配置类进行处理
                    } else {
                        this.importStack.registerImport(currentSourceClass.getMetadata(), candidate.getMetadata().getClassName());
                        this.processConfigurationClass(candidate.asConfigClass(configClass), exclusionFilter);
                    }
                }
            } catch (BeanDefinitionStoreException var17) {
                throw var17;
            } catch (Throwable var18) {
                throw new BeanDefinitionStoreException("Failed to process import candidates for configuration class [" + configClass.getMetadata().getClassName() + "]", var18);
            } finally {
                this.importStack.pop();
            }
        }

    }
}
```

虽然这里额外处理了`ImportSelector`对象，但是还针对`ImportSelector`的子接口`DeferredImportSelector`进行了额外处理，Deferred是延迟的意思，它是一个延迟执行的`ImportSelector`，并不会立即进处理，而是丢进DeferredImportSelectorHandler，并且在`parse`方法的最后进行处理：

```java
public void parse(Set<BeanDefinitionHolder> configCandidates) {
     ...

    this.deferredImportSelectorHandler.process();
}
```

`DeferredImportSelector`正好就有一个`process`方法：

```java
public interface DeferredImportSelector extends ImportSelector {
    @Nullable
    default Class<? extends DeferredImportSelector.Group> getImportGroup() {
        return null;
    }

    public interface Group {
        void process(AnnotationMetadata metadata, DeferredImportSelector selector);

        Iterable<DeferredImportSelector.Group.Entry> selectImports();

        public static class Entry {
          ...
```

最后经过ConfigurationClassParser处理完成后，通过`parser.getConfigurationClasses()`就能得到通过配置类导入了哪些额外的配置类。最后将这些配置类全部注册BeanDefinition，然后就可以交给接下来的Bean初始化过程去处理了。

```java
this.reader.loadBeanDefinitions(configClasses);
```

最后再看看`loadBeanDefinitions`是如何运行的：

```java
public void loadBeanDefinitions(Set<ConfigurationClass> configurationModel) {
    ConfigurationClassBeanDefinitionReader.TrackedConditionEvaluator trackedConditionEvaluator = new ConfigurationClassBeanDefinitionReader.TrackedConditionEvaluator();
    Iterator var3 = configurationModel.iterator();

    while(var3.hasNext()) {
        ConfigurationClass configClass = (ConfigurationClass)var3.next();
        this.loadBeanDefinitionsForConfigurationClass(configClass, trackedConditionEvaluator);
    }

}

private void loadBeanDefinitionsForConfigurationClass(ConfigurationClass configClass, ConfigurationClassBeanDefinitionReader.TrackedConditionEvaluator trackedConditionEvaluator) {
    if (trackedConditionEvaluator.shouldSkip(configClass)) {
        String beanName = configClass.getBeanName();
        if (StringUtils.hasLength(beanName) && this.registry.containsBeanDefinition(beanName)) {
            this.registry.removeBeanDefinition(beanName);
        }

        this.importRegistry.removeImportingClass(configClass.getMetadata().getClassName());
    } else {
        if (configClass.isImported()) {
            this.registerBeanDefinitionForImportedConfigurationClass(configClass);  //注册配置类自己
        }

        Iterator var3 = configClass.getBeanMethods().iterator();

        while(var3.hasNext()) {
            BeanMethod beanMethod = (BeanMethod)var3.next();
            this.loadBeanDefinitionsForBeanMethod(beanMethod); //注册@Bean注解标识的方法
        }

      	//注册`@ImportResource`引入的XML配置文件中读取的bean定义
        this.loadBeanDefinitionsFromImportedResources(configClass.getImportedResources());
     	 //注册configClass中经过解析后保存的所有ImportBeanDefinitionRegistrar，注册对应的BeanDefinition
        this.loadBeanDefinitionsFromRegistrars(configClass.getImportBeanDefinitionRegistrars());
    }
}
```

这样，整个`@Configuration`配置类的底层配置流程就大致了解了。

接着来看AutoConfigurationImportSelector是如何实现自动配置的，可以看到内部类AutoConfigurationGroup的process方法，它是父接口的实现，因为父接口是`DeferredImportSelector`，那么很容易得知，实际上最后会调用`process`方法获取所有的自动配置类：

```java
public void process(AnnotationMetadata annotationMetadata, DeferredImportSelector deferredImportSelector) {
    Assert.state(deferredImportSelector instanceof AutoConfigurationImportSelector, () -> {
        return String.format("Only %s implementations are supported, got %s", AutoConfigurationImportSelector.class.getSimpleName(), deferredImportSelector.getClass().getName());
    });
  	//获取所有的Entry，其实就是，读取spring.factories来查看有哪些自动配置类
    AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = ((AutoConfigurationImportSelector)deferredImportSelector).getAutoConfigurationEntry(annotationMetadata);
    this.autoConfigurationEntries.add(autoConfigurationEntry);
    Iterator var4 = autoConfigurationEntry.getConfigurations().iterator();

    while(var4.hasNext()) {
        String importClassName = (String)var4.next();
        this.entries.putIfAbsent(importClassName, annotationMetadata);
    }

}
```

接着来看`getAutoConfigurationEntry`方法：

```java
protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
  	//判断是否开启了自动配置，是的，自动配置可以关
    if (!this.isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    } else {
      	//根据注解定义获取一些属性
        AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
      	//得到spring.factories文件中所有需要自动配置的类
        List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
        ... 这里先看前半部分
    }
}
```

注意这里并不是spring.factories文件中所有的自动配置类都会被加载，它会根据@Condition注解的条件进行加载。这样就能实现我们需要什么模块添加对应依赖就可以实现自动配置了。

### 自定义Starter

仿照Mybatis来编写一个自己的starter，Mybatis的starter包含两个部分：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot</artifactId>
    <version>2.2.0</version>
  </parent>
  <!--  starter本身只做依赖集中管理，不编写任何代码  -->
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <name>mybatis-spring-boot-starter</name>
  <properties>
    <module.name>org.mybatis.spring.boot.starter</module.name>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <!--  编写的专用配置模块   -->
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-autoconfigure</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
    </dependency>
  </dependencies>
</project>
```

因此将我们自己的starter这样设计：

设计三个模块：

* spring-boot-hello：基础业务功能模块
* spring-boot-starter-hello：启动器
* spring-boot-autoconifgurer-hello：自动配置依赖

首先是基础业务功能模块，这里随便创建一个类就可以了：

```java
public class HelloWorldService {
	
}
```

启动器主要做依赖管理，这里就不写任何代码，只写pom文件：

```xml
<dependencies>
    <dependency>
        <groupId>org.example</groupId>
        <artifactId>spring-boot-autoconfigurer-hello</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

导入autoconfigurer模块作为依赖即可，接着去编写autoconfigurer模块，首先导入依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure</artifactId>
        <version>2.6.2</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <version>2.6.2</version>
        <optional>true</optional>
    </dependency>
    
    <dependency>
        <groupId>org.example</groupId>
        <artifactId>spring-boot-hello</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

接着创建一个HelloWorldAutoConfiguration作为自动配置类：

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication
@ConditionalOnClass(HelloWorldService.class)
@EnableConfigurationProperties(HelloWorldProperties.class)
public class HelloWorldAutoConfiguration {

    Logger logger = Logger.getLogger(this.getClass().getName());

    @Resource
    HelloWorldProperties properties;

    @Bean
    public HelloWorldService helloWorldService(){
        logger.info("自定义starter项目已启动！");
        logger.info("读取到自定义配置："+properties.getValue());
        return new HelloWorldService();
    }
}
```

对应的配置读取类：

```java
@ConfigurationProperties("hello.world")
public class HelloWorldProperties {

    private String value;

    public void setValue(String value) {
        this.value = value;
    }

    public String getValue() {
        return value;
    }
}
```

最后再编写`spring.factories`文件，并将我们的自动配置类添加即可：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.hello.autoconfigurer.HelloWorldAutoConfiguration
```

最后在Maven根项目执行`install`安装到本地仓库，完成。

这样就可以在其他项目中使用自己编写的starter了。

### Runner接口

在项目中，可能需要在项目启动完成之后，紧接着执行一段代码。

可以编写自定义的ApplicationRunner来解决，它会在项目启动完成后执行：

```java
@Component
public class TestRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("我是自定义执行！");
    }
}
```

当然也可以使用CommandLineRunner，它也支持使用@Order或是实现Ordered接口来支持优先级执行。

实际上它就是run方法的最后：

```java
public ConfigurableApplicationContext run(String... args) {
    ....

        listeners.started(context, timeTakenToStartup);
  			//这里已经完成整个SpringBoot项目启动，所以执行所有的Runner
        this.callRunners(context, applicationArguments);
    } catch (Throwable var12) {
        this.handleRunFailure(context, var12, listeners);
        throw new IllegalStateException(var12);
    }

    try {
        Duration timeTakenToReady = Duration.ofNanos(System.nanoTime() - startTime);
        listeners.ready(context, timeTakenToReady);
        return context;
    } catch (Throwable var11) {
        this.handleRunFailure(context, var11, (SpringApplicationRunListeners)null);
        throw new IllegalStateException(var11);
    }
}
```

## 再谈Spring框架

巩固和强化对于Spring框架的认识。

### 任务调度

为了执行某些任务，可能需要一些非常规的操作，比如希望使用多线程来处理结果或是执行一些定时任务，到达指定时间再去执行。

首先想到的就是创建一个新的线程来处理，或是使用TimerTask来完成定时任务

但是有了Spring框架之后，就不用这样了，因为Spring框架提供了更加便捷的方式进行任务调度。

#### 异步任务

需要使用Spring异步任务支持，在配置类上添加`@EnableAsync`或是在SpringBoot的启动类上添加也可以。

```java
@EnableAsync
@SpringBootApplication
public class SpringBootWebTestApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootWebTestApplication.class, args);
    }
}
```

接着，在需要异步执行的方法上，添加`@Async`注解即可将此方法标记为异步，当此方法被调用时，会异步执行（也就是新开一个线程执行，而不是在当前线程执行）

```java
@Service
public class TestService {

    @Async
    public void test(){
        try {
            Thread.sleep(3000);
            System.out.println("我是异步任务！");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
@RequestMapping("/login")
public String login(HttpServletRequest request){
    service.test();
    System.out.println("我是同步任务！");
    return "login";
}
```

实际上这也是得益于AOP机制，通过线程池实现。

但是也要注意，正是因为它是AOP机制的产物，所以它只能是在Bean中才会生效！

使用 @Async 注释的方法可以返回 'void' 或 "Future" 类型。Future是一种用于接收任务执行结果的一种类型（学到Java并发编程再说）

#### 定时任务

定时任务其实就是指定在哪个时候再去执行，在JavaSE阶段用TimerTask来执行定时任务。

Spring中的定时任务是全局性质的，当Spring程序启动后，那么定时任务也就跟着启动了。

在配置类上添加`@EnableScheduling`或是在SpringBoot的启动类上添加也可：

```java
@EnableAsync
@EnableScheduling
@SpringBootApplication
public class SpringBootWebTestApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootWebTestApplication.class, args);
    }
}
```

接着创建一个定时任务配置类，在配置类里面编写定时任务：

```java
@Configuration
public class ScheduleConfiguration {

    @Scheduled(fixedRate = 2000)
    public void task(){
        System.out.println("我是定时任务！"+new Date());
    }
}
```

`@Scheduled`中有很多参数，需要指定'cron', 'fixedDelay(String)', or 'fixedRate(String)'的其中一个，否则无法创建定时任务

* fixedDelay：在上一次定时任务执行完之后，间隔多久继续执行。
* fixedRate：无论上一次定时任务有没有执行完成，两次任务之间的时间间隔。
* cron：使用cron表达式来指定任务计划。

了解一下cron表达式：https://blog.csdn.net/sunnyzyq/article/details/98597252

### 监听器

监听实际上就是等待某个事件的触发，当事件触发时，对应事件的监听器就会被通知。

通过监听事件，我们就可以在对应的时机进行一些额外的处理

```java
@Component
public class TestListener implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        System.out.println(event.getApplicationContext());
    }
}
```

通过阅读源码得知，一个事件实际上就是通过`publishEvent`方法来进行发布的。我们也可以自定义自己项目中的事件，并注册对应的监听器进行处理。

比如以下代码实现了自定义事件发布和监听。

```java
public class TestEvent extends ApplicationEvent {   //需要继承ApplicationEvent
    public TestEvent(Object source) {
        super(source);
    }
}
```

```java
@Component
public class TestListener implements ApplicationListener<TestEvent> {

    @Override
    public void onApplicationEvent(TestEvent event) {
        System.out.println("自定义事件发生了："+event.getSource());
    }
}
```

```java
@Resource
ApplicationContext context;

@RequestMapping("/login")
public String login(HttpServletRequest request){
    context.publishEvent(new TestEvent("有人访问了登录界面！"));
    return "login";
}
```

### Aware系列接口

在之前查看Spring源码时，经常会发现某些类的定义上，除了一些继承关系以外，还实现了一些接口，他们的名称基本都是`xxxxAware`。
 
比如SpringSecurity的源码，AbstractAuthenticationProcessingFilter类就是这样：

它除了继承自GenericFilterBean之外，还实现了ApplicationEventPublisherAware和MessageSourceAware接口

```java
public abstract class AbstractAuthenticationProcessingFilter extends GenericFilterBean implements ApplicationEventPublisherAware, MessageSourceAware {
    protected ApplicationEventPublisher eventPublisher;
    protected AuthenticationDetailsSource<HttpServletRequest, ?> authenticationDetailsSource = new WebAuthenticationDetailsSource();
    private AuthenticationManager authenticationManager;
    ...
```

Aware的中文意思为**感知**。简单来说，它就是一个标识，实现此接口的类会获得某些感知能力。Spring容器会在Bean被加载时，根据类实现的感知接口，会调用类中实现的对应感知方法。

比如AbstractAuthenticationProcessingFilter就实现了ApplicationEventPublisherAware接口。此接口的感知功能为事件发布器，在Bean加载时，会调用实现类中的`setApplicationEventPublisher`方法，而AbstractAuthenticationProcessingFilter类则利用此方法，在Bean加载阶段获得了容器的事件发布器，以便之后发布事件使用。

```java
public void setApplicationEventPublisher(ApplicationEventPublisher eventPublisher) {
    this.eventPublisher = eventPublisher;   //直接存到成员变量
}
```

```java
protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
    SecurityContext context = SecurityContextHolder.createEmptyContext();
    context.setAuthentication(authResult);
    SecurityContextHolder.setContext(context);
    if (this.logger.isDebugEnabled()) {
        this.logger.debug(LogMessage.format("Set SecurityContextHolder to %s", authResult));
    }

    this.rememberMeServices.loginSuccess(request, response, authResult);
  	//在这里使用
    if (this.eventPublisher != null) {
        this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));
    }

    this.successHandler.onAuthenticationSuccess(request, response, authResult);
}
```

再比如BeanNameAware就是感知Bean名称的一个接口，当Bean被加载时，会调用`setBeanName`方法并将Bean名称作为参数传递。

```java
@Service
public class TestService implements BeanNameAware {
    @Override
    public void setBeanName(String s) {
        System.out.println(s);
    }
}
```

有关所有的Aware这里就不一一列举了。

