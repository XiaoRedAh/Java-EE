# Spring 注解开发

## 前置

**导入依赖**
导入Spring，AOP和Lombok
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.13</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>5.3.13</version>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.22</version>
    <scope>provided</scope>
</dependency>
```

**先创建一个注册类`@Configuration`**

注册类（等价于配置文件的文件头）
```java
@Configuration
public class MainConfiguration {
    //等价于配置文件的文件头
}
```

**到主函数那里创建容器**

```java
public static void main(String[] args) {
        AnnotationConfigApplicationContext context=new AnnotationConfigApplicationContext(MainConfiguration.class);
    }
```

## Bean注册

### `@bean`注册

**在注册类里面注册，返回值就是bean对象的类型**
bean对象默认别名是函数名

```java
@Configuration
public class MainConfiguration {
    @Bean("lbwnb")//默认别名是函数名"card"
    @Scope("prototype")//默认是单例模式
    Card card(){
        return new Card();
    }

    @Bean
    Student student(){
        //可以在注册的时候进行赋值操作（等价于xml的<poperty>）
        Student student=new Student();
        student.setAge(10);
        return student;
    }
}
```

**在主函数中获取bean（按类或按别名）**
按类获取，不用强制类型转换
按别名获取，需要强制类型转换

```java
Card card= (Card) context.getBean("lbwnb");
Student stu=context.getBean(Student.class);
```

### `@Component`+`@ComponentScan`注册

**注册类添加`@ComponentScan`**

```java
@Configuration
//扫多个包用@ComponentScans
@ComponentScans({@ComponentScan("com.test.bean"),
                @ComponentScan("com.test.aop")
})//扫描对应包下的所有类，有@Component的就注册
public class MainConfiguration {

}
```
   
**要注册的类上方添加`@Component`**
bean对象的别名默认是首字母小写的类名

```java
@Component("sss")//可以不起别名，默认是首字母小写的类名
@Scope("prototype")//默认单例模式
public class Student {
    String name;
    int age;
}
```

**在主函数中获取bean（按类或按别名）**
按类获取，不用强制类型转换
按别名获取，需要强制类型转换

```java
Card card= (Card) context.getBean("lbwnb");
Student stu=context.getBean(Student.class);
```

## 依赖注入

* @Resource默认**ByName**如果找不到则**ByType**，可以添加到set方法、字段上。
* @Autowired默认是**byType**，可以添加在构造方法、set方法、字段、**方法参数**上。


### `@Resource`注入

**直接注入**
直接在类定义的属性上方`@Resource`,那么就会在构造函数的时候自动注入对应的bean对象

```java
@Component
public class Student {
    String name;
    int age;
    @Resource
    Card card;
}
```

**通过set方法注入**
在set函数上方`@Resource`，函数传入的参数自动注入对应的bean对象

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
### `@Autowire`注入

IDEA不推荐将其使用在字段上，会出现黄标。

默认匹配规则
Spring容器首先查询对应类型的Bean对象，如果只有一个，则直接装配给该属性/该`set`方法的参数；如果同类型的Bean对象不止一个，则以变量名称作为id查找同类对象

## 构造后执行和销毁前执行的方法

等价于xml的init—method和destroy-method
构造后执行：`@PostConstruct`      
销毁前执行：`@PreDestroy`

**先调用构造方法，再调用注入方法，最后调用构造后的初始化方法**

```java
@Component
public class Student {
    String name;
    int age;
    @Resource
    Card card;
    Student(){
        System.out.println("我是构造方法");
    }
    @Resource
    public void setCard(Card card) {
        System.out.println("我是通过set方法注入的属性");
        this.card = card;
    }
    @PostConstruct
    public void init(){
        System.out.println("我是构造后的初始化方法");
    }
    @PreDestroy
    public void destroy(){
        System.out.println("我是销毁前的方法");
    }
}
```

## Aop操作

首先创建一个专门作Aop操作的包，在里面创建一个`AopTest`类
这个类需要注册为Bean，注册类需添加`@EnableAspectJAutoProxy`

```java
@Configuration
@ComponentScans({@ComponentScan("com.test.bean"),
                @ComponentScan("com.test.aop")
})
@EnableAspectJAutoProxy
public class MainConfiguration {

}
```

**Aop操作三要素：`JoinPoint`，`Advice`，`Point Cut`**

* `JoinPoint`:包含了切入点的信息
* `Advice`：用来写代码的，就是说明切上去之后要干嘛
>`@Before`:该方法在切入点方法调用之前调用
`@AfterReturning`：该方法在切入点调用之后调用
`@Around`：前两者的综合，切入点方法调用的时机在环绕方法里确定

* `Point Cut`:用来定义这些代码是切在哪些位置
>**execution:**  Join Point方法签名满足特定条件
**@annotation：**   Join Point方法用特定注解标注（比较灵活，不需要切入点有什么共同特征，想要切入的地方打个标签就行）

**在Aop类上方添加`@Aspect`，并写方法**

```java
@Aspect
@Component
public class AopTest {
    @Before("execution(* com.test.bean.Student.say(String))")
    public void before(){
        System.out.println("我是say方法调用前执行的内容");
    }
    @AfterReturning(value="execution(* com.test.bean.Student.say(String))")
    public void after(){
        System.out.println("我是say方法调用后执行的内容");
    }
}
```

可以给方法添加JointPoint参数获得切入点信息

```java
@Before("execution(* com.test.bean.Student.say(String))")
    public void before(JoinPoint point){
        System.out.println("我是say方法调用前执行的内容");
        System.out.println("say方法传入的参数是："+ Arrays.toString(point.getArgs()));
    }
```

`@AfterReturning`可以获切入点的返回值

``` java
 @AfterReturning(value="execution(* com.test.bean.Student.say(String))",returning = "val")
    public void after(JoinPoint point,Object val){
        System.out.println("我是say方法调用后执行的内容");
        System.out.println("调用say方法的对象是："+point.getThis());
        System.out.println("say方法的返回值是："+val);
    }
```

`@Around`环绕方法需要帮助切入点方法返回其函数返回值，否则切入点方法只能返回null

```java
  @Around("execution(* com.test.bean.Student.say(String))")
    public Object around(ProceedingJoinPoint point)throws Throwable{
        System.out.println("我是say方法调用后执行的内容");
        Object val=point.proceed();//执行切入点方法，并记录返回值
        System.out.println("我是say方法调用后执行的内容");
        return val;//环绕方法需要帮助切入点方法返回其函数返回值，不然切入点方法只能返回null
    }
```

## 其他操作
`@Import`

## 整合Mybatis

**不再需要Mybatis配置文件**

**配置类中**

1. 首先创建一个数据源的实现类（相当于配置文件）
2. 然后将这个数据源给到`SqlSessionFactoryBean`实例，

这样，就相当于直接在一开始通过IoC容器配置了`SqlSessionFactory`，只需要传入一个`DataSource`的实现即可。

```java
@Configuration
@ComponentScan("com.test")
@MapperScan("com.test.mapper")
public class TestConfiguration {
    @Bean
    public DataSource dataSource() throws SQLException {
      HikariDataSource dataSource = new HikariDataSource();
      dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/study");
      dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
      dataSource.setUsername("root");
      dataSource.setPassword("cronaldo7");
      return dataSource;
   }

    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(@Autowired DataSource dataSource){
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        return bean;
    }
}
```

**mapper包**

`TestMapper`接口类

```java
public interface TestMapper {
    @Select("select * from student where sid= '110'")
    Student getStudent();

}
```

**service包**

`TestService`接口类

```java
public interface TestService {
    Student getStudent();
}
```

`TestServiceImpl`接口实现类

`@Resource`注入`TestMapper`的bean对象，使用这个映射类里面的`getStudent`实现接口

```java
@Component
public class TestServiceImpl implements TestService {
    @Resource
    TestMapper mapper;

    @Override
    public Student getStudent() {
        return mapper.getStudent();
    }
}
```

**Main函数**

删除配置文件，同样可以正常使用Mapper。

而且这里获取的是`TestService`接口的Bean对象

```java
public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context=new AnnotationConfigApplicationContext(MainConfiguration.class);
        TestService service=context.getBean(TestService.class);
        System.out.println(service.getStudent());
    }
}
```


