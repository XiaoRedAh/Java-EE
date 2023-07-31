# SpringBoot一站式开发

官网：https://spring.io/projects/spring-boot

 Spring Boot可以轻松创建独立的、基于Spring的生产级应用程序，它可以让你“运行即可”。大多数Spring Boot应用程序只需要少量的Spring配置。

在SSM阶段，当需要搭建一个基于Spring全家桶的Web应用程序时，不得不做大量的依赖导入和框架整合相关的Bean定义，光是整合框架就花费了大量的时间。

但是实际上，整合框架其实基本都是一些固定流程，每创建一个新的Web应用程序，基本都会使用同样的方式去整合框架。因此我们完全可以将一些重复的配置作为约定，只要框架遵守这个约定，提供默认的配置，这样就不用我们再去配置了。

而SpringBoot正是将这些过程大幅度进行了简化，它可以自动进行配置，只需要导入对应的启动器（starter）依赖即可。

SpringBoot功能：

- 创建独立的Spring应用程序
- 直接嵌入Tomcat、Jetty或Undertow（无需部署WAR包，打包成Jar本身就是一个可以运行的应用程序）
- 提供一站式的“starter”依赖项，以简化Maven配置（需要整合什么框架，直接导对应框架的starter依赖）
- 尽可能自动配置Spring和第三方库（除非特殊情况，否则几乎不需要你进行什么配置）
- 提供生产就绪功能，如指标、运行状况检查和外部化配置
- 没有代码生成，也没有XML配置的要求

# SpringBoot两大特性

## 起步依赖

**依赖冲突**

一般程序在运行时发生类似于 java.lang.ClassNotFoundException，Method not found: '……'，或者莫名其妙的异常信息，这种情况一般很大可能就是 jar包依赖冲突的问题引起的了。

原因：

​一般在是A依赖C(低版本C)，B也依赖C(高版本C)。 都是他们依赖的又是不同版本的C的时候会出现。

**解决依赖冲突**

排除掉低版本的依赖，留下高版本的依赖。

借助MavenHelper插件来帮助排查

**SpringBoot的版本锁定帮助我们解决依赖冲突**

SpringBoot模块都需要继承一个父工*spring-boot-starter-parent*。在spring-boot-starter-parent的父工程*spring-boot-dependencies*中对常用的依赖进行了版本锁定。这样我们在添加依赖时，很多时候都不需要添加依赖的版本号了。

**starter机制**

需要使用某种功能时只需要引入对应的starter即可。一个starter针对一种特定的场景，其内部引入了该场景所需的依赖，这样就不需要单独引入多个依赖了。

​命名规律

一般来说，非官方starter是没有进行版本锁定的，所以要指定版本号

- 官方starter都是以  `spring-boot-starter`开头后面跟上场景名称。例如：spring-boot-starter-data-jpa
- 非官方starter则是以 `场景名-spring-boot-starter`的格式，例如：mybatis-spring-boot-starter


## 自动配置

**概念**

Springboot遵循“**约定优于配置**”的原则，自动进行了默认配置。

需要使用什么场景时，就会自动配置这个场景相关的配置。

​如果默认配置不符合需求，修改这部分配置即可。

**原理**

等到看源码的时候再说

# 前置—检查环境

Jdk：8
Maven：3.5以上（“文件-设置-Maven”查看）
SpringBoot：2.5以上，3.0以下

**注意**：如果maven无法解析依赖项，外部库总是导不入maven的jar包，则需打开maven的setting.xml文件（右击pom.xml-maven-打开setting.xml）添加以下内容：

```xml
<mirrors
        <mirror
            <idaliyunmaven</id
            <mirrorOfcentral</mirrorOf
            <namealiyun maven</name
            <urlhttps://maven.aliyun.com/repository/public </url
        </mirror
    </mirrors
    <profiles
        <profile
            <idjdk-1.8</id
            <activation
                <activeByDefaulttrue</activeByDefault
                <jdk1.8</jdk
            </activation
            <properties
                <maven.compiler.source1.8</maven.compiler.source
                <maven.compiler.target1.8</maven.compiler.target
                <maven.compiler.compilerVersion1.8</maven.compiler.compilerVersion
            </properties
        </profile
    </profiles
```

# SpringBoot项目文件结构（啥都不勾选）

**自带一个SpringBoot启动类**

创建SpringBoot项目之后，首先会自动生成一个主类，而主类中的`main`方法中调用了`SpringApplication`类的静态方法`run()`来启动整个SpringBoot项目，并且主类的上方有一个`@SpringBootApplication`注解：

```java
@SpringBootApplication
public class SpringBootTestApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootTestApplication.class, args);
    }

}
```

**同时还自带了一个测试类**

测试类的上方仅添加了一个`@SpringBootTest`注解：

```java
@SpringBootTest
class SpringBootTestApplicationTests {

    @Test
    void contextLoads() {
        
    }

}
```

**pom.xml文件**

```xml
<?xml version="1.0" encoding="UTF-8"?
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd"
    <modelVersion4.0.0</modelVersion
  	<!--  继承父工程  --
    <parent
        <groupIdorg.springframework.boot</groupId
        <artifactIdspring-boot-starter-parent</artifactId
        <version2.6.2</version
        <relativePath/ <!-- lookup parent from repository --
    </parent
    <groupIdcom.example</groupId
    <artifactIdspringboot-study</artifactId
    <version0.0.1-SNAPSHOT</version
    <nameSpringBootTest</name
    <descriptionSpringBootTest</description
    <properties
        <java.version1.8</java.version
    </properties
    <dependencies
        <!--  spring-boot-starter SpringBoot核心启动器  --
        <dependency
            <groupIdorg.springframework.boot</groupId
            <artifactIdspring-boot-starter</artifactId
        </dependency
        <!--  spring-boot-starter-test SpringBoot测试模块启动器  --
        <dependency
            <groupIdorg.springframework.boot</groupId
            <artifactIdspring-boot-starter-test</artifactId
            <scopetest</scope
        </dependency
    </dependencies

    <build
        <plugins
            <!--  SpringBoot Maven插件，打包Jar都不用你操心了   --
            <plugin
                <groupIdorg.springframework.boot</groupId
                <artifactIdspring-boot-maven-plugin</artifactId
            </plugin
        </plugins
    </build

</project
```

**除了以上这些文件以外，项目目录下还有**

* .gitignore  -  Git忽略名单
* application.properties   -    SpringBoot的配置文件，所有依赖的配置都在这里编写，但是一般情况下只需要配置必要项即可。

# SpringBoot的配置文件

`application.properties`是整个SpringBoot的配置文件，除了使用`properties`格式以外，还有一种叫做`yaml`格式，SringBoot支持该格式

YML和properties格式的相互转换：https://www.toyaml.com/index.html

想要在编写配置文件的时候有相应的提示，可以添加以下依赖

```xml
<dependency
    <groupIdorg.springframework.boot</groupId
    <artifactIdspring-boot-configuration-processor</artifactId
    <optionaltrue</optional
</dependency
```

## yml基本语法

**什么是YML（或YMAL）**

YAML (YAML Ain't a Markup Language)YAML不是一种标记语言，以.yml为后缀的文件，是一种直观的能够被电脑识别的数据序列化格式，并且容易被人类阅读，容易和脚本语言交互的，可以被支持YAML库的不同的编程语言程序导入，专门用来写配置文件的语言。

yml文件比xml更加简洁易读：

yml

```yml
student:
    name: sangeng
    age: 15
```

在xml里这样表示

```xml
<student
    <namesangeng</name
    <age15</age
</student
```

**约定**

- k: v 表示键值对关系，**冒号后面必须有一个空格**

- 使用空格的缩进表示层级关系，空格数目不重要，**只要是左对齐的一列数据，都是同一个层级的**

- 大小写敏感

- 缩进时**不允许使用Tab键，只允许使用空格**。

- java中对于驼峰命名法，可用原名或使用-代替驼峰，如java中的lastName属性,在yml中使用lastName或 last-name都可正确映射。

- yml中注释前面要加#

**普通值(字面量)**

键名: 值

字符串默认不用加上单引号或者双引号

双引号:转义字符能够起作用(换行，...)

''：单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据

```yml
name1: sangeng 
name2: 'sangeng  \n caotang'
name3: "sangeng  \n caotang"
age: 15
flag: true
```

**日期**

用/划分年月日

```yml
date: 2019/01/01
```

**对象(属性和值)、Map(键值对)**

多行写法：

在下一行来写对象的属性和值的关系，注意缩进 

```yml
student:
  name: zhangsan
  age: 20
```

行内写法：

```yml
student: {name: zhangsan,age: 20}
```

**数组、list、set**

用- 值表示数组中的一个元素 

多行写法：

```yml
pets:
  - dog
  - pig
  - cat
```

行内写法：

```yml
pets: [dog,pig,cat]
```



**对象数组、对象list、对象set**

可以多行写也可以行内写，比如下面这个例子，前两个对象是多行写法，最后一个对象是行内写法

```yml
students:
 - name: zhangsan
   age: 22
 - name: lisi
   age: 20
 - {name: wangwu,age: 18}
```

**占位符赋值**

使用 **${key:defaultValue}** 的方式来赋值，若key不存在，则会使用defaultValue来赋值。

例如

服务器的端口号会被自定义的myPort赋值，如果myPort没有，则是默认的88

```yml
server:
  port: ${myPort:88}

myPort: 80   
```

## SpringBoot读取配置文件

## `@Value`注入配置文件的值

*注意：加了@Value的类必须是交由Spring容器管理的*

​使用此注解只能获取简单类型的值（8种基本数据类型及其包装类，String,Date）

例如

`MainController`类的属性data就会注入配置文件里的test.data的值，也就是data=100

```yml
test：
  data: 100
```

```java
@Controller
public class MainController {

    @Value("${test.data}")
    int data;
```

## `@ConfigurationProperties`注入

*注意：对应的属性要有set/get方法，并且key要和成员变量名一致才可以对应上*

​yml配置

```yml
student:
  lastName: sangeng
  age: 17
student2:
  lastName: sangeng2
  age: 15
```

​在类上添加注解`@Component`和`@ConfigurationProperties(prefix = "配置前缀")`

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Component
@ConfigurationProperties(prefix = "student")
public class Student {
    private String lastName;
    private Integer age;
}
```

​从spring容器中获取Student对象

这时会发现这个对象的lastName是“sangeng”，age是17

```java
@RestController
public class TestController {

    @Autowired
    private Student student;
    @RequestMapping("/test")
    public String test(){
        System.out.println(student);
        return "hi";
    }
}
```

# Web开发

## 整合Web相关框架

SpringBoot会内嵌一个Tomcat服务器，也就是当Jar打包后，相当于就是一个可以直接运行的应用程序

**创建一个基于SpringBoot的Web应用程序**

只需导入web的starter（可以看到它包含了tomcat，json,SpringMVC等依赖）【也可以在创建项目的时候勾选“Spring Web”】

```xml
<dependency
        <groupIdorg.springframework.boot</groupId
        <artifactIdspring-boot-starter-web</artifactId
</dependency
```

**它是真的快**

创建完成后，直接开启项目，就可以直接访问：http://localhost:8080/ 。由于还没有编写任何的请求映射，所以没有数据。

看看日志：

日志中除了最基本的SpringBoot启动日志以外，还新增了内嵌Web服务器（Tomcat）的启动日志，并且显示了当前Web服务器所开放的端口，并且自动帮助我们初始化了DispatcherServlet（现在只是创建了项目，导入了web相关的starter依赖，没有进行任何的配置，实际上它使用的是starter提供的默认配置进行初始化的）

```
2022-01-06 22:17:46.308  INFO 853 --- [           main] c.example.SpringBootWebTestApplication   : Starting SpringBootWebTestApplication using Java 1.8.0_312 on NagodeMacBook-Pro.local with PID 853 (/Users/nagocoler/Downloads/SpringBootWebTest/target/classes started by nagocoler in /Users/nagocoler/Downloads/SpringBootWebTest)
2022-01-06 22:17:46.309  INFO 853 --- [           main] c.example.SpringBootWebTestApplication   : No active profile set, falling back to default profiles: default
2022-01-06 22:17:46.629  INFO 853 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2022-01-06 22:17:46.632  INFO 853 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2022-01-06 22:17:46.632  INFO 853 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.56]
2022-01-06 22:17:46.654  INFO 853 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2022-01-06 22:17:46.654  INFO 853 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 325 ms
2022-01-06 22:17:46.780  INFO 853 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2022-01-06 22:17:46.785  INFO 853 --- [           main] c.example.SpringBootWebTestApplication   : Started SpringBootWebTestApplication in 0.62 seconds (JVM running for 0.999)
2022-01-06 22:18:02.979  INFO 853 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2022-01-06 22:18:02.979  INFO 853 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2022-01-06 22:18:02.980  INFO 853 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 1 ms
```

由于SpringBoot是自动扫描的，因此直接创建一个Controller即可被加载：

现在几乎没有做任何配置，但是可以直接开始配置Controller

*注意：Controller一定要在启动类所在的包或者其子包下（否则会404，原因学到源码再说）*

```java
@Controller
public class MainController {

  	//直接访问http://localhost:8080/index即可，不用加web应用程序名称了
    @RequestMapping("/index")
    @ResponseBody
    public String index(){
        return "你好，欢迎访问主页！";
    }
}
```

它还可以自动识别类型，如果返回的是一个对象类型的数据，那么它会自动转换为JSON数据格式，无需配置：

```java
@Data
public class Student {
    int sid;
    String name;
    String sex;
}
```

```java
@RequestMapping("/student")
@ResponseBody
public Student student(){
    Student student = new Student();
    student.setName("小明");
    student.setSex("男");
    student.setSid(10);
    return student;
}
```

最后浏览器能够直接得到`application/json`的响应数据

## 整合SpringSecurity依赖

只需要导入SpringSecurity的Starter依赖即可：

```xml
<dependency
    <groupIdorg.springframework.boot</groupId
    <artifactIdspring-boot-starter-security</artifactId
</dependency
```

SpringSecurity会自动生成一个默认用户`user`，它的密码会出现在日志中：

```
2022-01-06 23:10:51.329  INFO 2901 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2022-01-06 23:10:51.329  INFO 2901 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.56]
2022-01-06 23:10:51.350  INFO 2901 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2022-01-06 23:10:51.351  INFO 2901 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 341 ms
2022-01-06 23:10:51.469  INFO 2901 --- [           main] .s.s.UserDetailsServiceAutoConfiguration : 

Using generated security password: ff24bee3-e1b7-4309-9609-d32618baf5cb

```

`ff24bee3-e1b7-4309-9609-d32618baf5cb`就是随机生成的一个密码，可以使用此用户登录。

可以在配置文件中直接配置(实际上就是一个`inMemoryAuthentication`)：

```yaml
spring:
  security:
    user:
      name: test   # 用户名
      password: 123456  # 密码
      roles:   # 角色
      - user
      - admin
```

页面的控制和数据库验证还是需要提供`WebSecurityConfigurerAdapter`的实现类去完成：

*注意：这里不需要再添加`@EnableWebSecurity`了，因为starter依赖已经帮我们添加了。*

```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/login").permitAll()
                .anyRequest().hasRole("user")
                .and()
                .formLogin();
    }
}
```

使用了SpringBoot之后，需要什么功能，只需要导入对应的starter依赖即可，甚至都不需要去进行额外的配置，只需要关注依赖本身的必要设置即可。

## 整合junit

使用SpringBoot整合Junit进行单元测试。

Spring Boot 2.2.0 版本开始引入 JUnit 5 作为单元测试默认库

## 整合流程

①添加依赖

```xml
        <dependency
            <groupIdorg.springframework.boot</groupId
            <artifactIdspring-boot-starter-test</artifactId
        </dependency
```

②编写测试类

test.java.com.sangeng.ApplicationTest

```java
import com.sangeng.controller.HelloController;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class ApplicationTest {

    @Autowired
    private HelloController helloController;

    @Test
    public void testJunit(){
        System.out.println(1);
        System.out.println(helloController);
    }
}
```

*注意：测试类所在的包需要和启动类是在同一个包下。否则就要使用如下写法指定启动类*

```java
import com.sangeng.controller.HelloController;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

//classes属性来指定启动类
@SpringBootTest(classes = HelloApplication.class)
public class ApplicationTest {

    @Autowired
    private HelloController helloController;

    @Test
    public void testJunit(){
        System.out.println(1);
        System.out.println(helloController);
    }
}
```

## 兼容老版本

由于Junit5和之前的Junit4有比较大的不同，如果对老项目中的SpringBoot进行了版本升级，那之前的单元测试代码会出现一些问题

![img](img\junit5.jpeg) 

从上图可以看出**JUnit 5 = JUnit Platform+JUnit Jupiter+JUnit Vintage**

- **JUnit Platform**： 这是Junit提供的平台功能模块，通过它，其它的测试引擎也可以接入
- **JUnit JUpiter**：这是JUnit5的核心，是一个基于JUnit Platform的引擎实现，它包含许多丰富的新特性来使得自动化测试更加方便和强大。
- **JUnit Vintage**：这个模块是兼容JUnit3、JUnit4版本的测试引擎，使得旧版本的自动化测试也可以在JUnit5下正常运行。

虽然Junit5包含了**JUnit Vintage**来兼容JUnit3和Junit4，但是**SpringBoot 2.4 以上版本对应的spring-boot-starter-test移除了默认对Vintage 的依赖**。

所以仅仅依赖spring-boot-starter-test时会发现之前使用的@Test注解和@RunWith注解都不能使用了。

**解决兼容问题**

添加依赖vintage来进行兼容。

```xml
        <dependency
            <groupIdorg.junit.vintage</groupId
            <artifactIdjunit-vintage-engine</artifactId
            <scopetest</scope
        </dependency
```

*注意：org.junit.Test对应的是Junit4的版本，就搭配@RunWith注解来使用*

SpringBoot2.2.0之前版本的写法

```java
import com.sangeng.controller.HelloController;
//import org.junit.jupiter.api.Test;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

//classes属性来指定启动类
@SpringBootTest
@RunWith(SpringRunner.class)
public class ApplicationTest {

    @Autowired
    private HelloController helloController;

    @Test
    public void testJunit(){
        System.out.println(1);
        System.out.println(helloController);
    }
}
```

## 整合Mybatis框架

**先准备用于测试的数据**

①数据准备

```mysql
/*Table structure for table `user` */
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `address` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8;

/*Data for the table `user` */

insert  into `user`(`id`,`username`,`age`,`address`) values (2,'pdd',25,'上海'),(3,'UZI',19,'上海11'),(4,'RF',19,NULL),(6,'三更',14,'请问2'),(8,'test1',11,'cc'),(9,'test2',12,'cc2');

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;
```

②实体类

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer id;
    private String username;
    private Integer age;
    private String address;
}
```

整合Mybatis流程：

**①导入对应的starter依赖**

具体用哪个版本的Mybatis要看配置: https://github.com/mybatis/spring-boot-starter/

```xml
<!--mybatis启动器--
<dependency
    <groupIdorg.mybatis.spring.boot</groupId
    <artifactIdmybatis-spring-boot-starter</artifactId
    <version2.2.0</version
</dependency
        <!--mysql驱动--
<dependency
    <groupIdmysql</groupId
    <artifactIdmysql-connector-java</artifactId
    <version8.0.27</version
    <scoperuntime</scope
</dependency
```

**②配置`application.yml`文件**

1. spring配置数据源
2. mybatis配置映射文件位置

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mybatis_db?characterEncoding=utf-8&serverTimezone=UTC&&useSSL=false # 第一个参数是数据库名称
    username: root
    password: cronaldo7
    driver-class-name: com.mysql.cj.jdbc.Driver
mybatis:
  mapper-locations: classpath:mapper/*Mapper.xml # mapper映射文件路径，说明映射文件在resource/mapper包下，文件名都以Mapper.xml结尾
  type-aliases-package: com.sangeng.domain   # 配置哪个包下的类有默认的别名
```

**③编写mapper接口，需要添加`@Mapper`和`@Repository`注解（可以不加）**

导入依赖后，就不需要再去设置Mybatis的相关Bean了，也不需要添加任何`@MapperSacn`注解，因为starter已经帮助我们做了，它会自动扫描项目中添加了`@Mapper`注解的接口，直接将其注册为Bean，不需要进行任何配置。

*当然，如果觉得每个接口都去加一个`@Mapper`比较麻烦的话，也可直接`@MapperScan`使用包扫描。*

**④编写mapper接口对应的xml文件**

## 整合Thymeleaf框架

整合Thymeleaf也只需导入对应的starter：

```xml
<dependency
    <groupIdorg.springframework.boot</groupId
    <artifactIdspring-boot-starter-thymeleaf</artifactId
</dependency
```

定义Controller

在controller中往域中存数据，并且跳转

```java
@Controller
public class ThymeleafController {

    @Autowired
    private UserServcie userServcie;

    @RequestMapping("/thymeleaf/users")
    public String users(Model model){
        //获取数据
        List<User users = userServcie.findAll();
        //望域中存入数据
        model.addAttribute("users",users);
        model.addAttribute("msg","hello thymeleaf");
        //页面跳转
        return "table-standard";
    }
}
```

注意:这样只能正常解析HTML页面（放在Resource下的templates目录），但是js、css等静态资源（Resource下的static目录）需要进行路径指定，不然无法访问

在application.yml文件里配置静态资源的访问前缀：

```yaml
spring:
	mvc:
  	static-path-pattern: /static/**
```

也可以在使用thymeleaf的html文件里这么写：

1. href,src都加上th:
2. 资源路径加上@{}，以/开头
   @{/app/css/bootstrap.css}就是引入resource/static/app/css/bootstrap.css

```html
   <link rel="stylesheet" th:href="@{/app/css/bootstrap.css}"
   <!-- Vendor CSS--
   <link rel="stylesheet" th:href="@{/vendor/fontawesome/css/font-awesome.min.css}"
   <link rel="stylesheet" th:href="@{/vendor/animo/animate+animo.css}"
   <link rel="stylesheet" th:href="@{/vendor/csspinner/csspinner.min.css}"
   <!-- START Page Custom CSS--
   <!-- END Page Custom CSS--
   <!-- App CSS--
   <link rel="stylesheet" th:href="@{/app/css/app.css}"
   <!-- Modernizr JS Script--
   <script th:src="@{/vendor/modernizr/modernizr.js}" type="application/javascript"</script
   <!-- FastClick for mobiles--
   <script th:src="@{/vendor/fastclick/fastclick.js}" type="application/javascript"</script
```

**简单使用Thymeleaf**

在**resources\templates**下存放模板页面,htmL文件添加：

```xml
<html lang="en" xmlns:th=http://www.thymeleaf.org
xmlns:sec=http://www.thymeleaf.org/extras/spring-security
```

获取域中的name属性的值可以使用： ${name},注意要在th开头的属性中使用

```html
<html lang="en" class="no-ie" xmlns:th="http://www.thymeleaf.org"
 .....
 <div class="panel-heading" th:text="${msg}"Kitchen Sink</div
```

遍历语法：

th:each="自定义的元素变量名称 : ${集合变量名称}" 

```html
<tr th:each="user:${users}"
    <td th:text="${user.id}"</td
    <td th:text="${user.username}"</td
    <td th:text="${user.age}"</td
    <td th:text="${user.address}"</td
</tr
```

## 访问静态资源

​由于SpringBoot的项目是打成jar包的，所以没有之前web项目的那些web资源目录(webapps)。

官方文档规定把静态资源放到 `resources/static`(或者 `resources/public`或者`resources/resources`或者`resources/META-INF/resources`) 中

​静态资源放完后，

访问文件：resources/static/index.html  只需要在访问时资源路径写成/index.html即可。  

​访问文件：resources/static/pages/login.html  访问的资源路径写成： /pages/login.html

**修改静态资源访问路径**

​SpringBoot默认的静态资源路径匹配为/** 。

可以通过`spring.mvc.static-path-pattern` 这个配置进行修改。

​例如想让访问静态资源的url必须有前缀/res才能访问到static目录中的资源，那么可以修改如下：

在application.yml中(/** 表示对当前目录及其子目录生效)

```yml
spring:
  mvc:
    static-path-pattern: /res/** #修改静态资源访问路径
```

访问文件：resources/static/index.html  需要在访问时资源路径写成/res/index.html

**修改静态资源存放目录**

通过`spring.web.resources.static-locations`这个配置来修改静态资源的存放目录。

​例如:
静态资源就会放在resources/sgstatic和resources/static/这两个目录下

```yml
spring:
  web:
    resources:
      static-locations:
        - classpath:/sgstatic/ 
        - classpath:/static/
```

## 响应格式统一

要保证一个项目中所有接口返回的数据格式的统一。这样无论是前端还是移动端开发获取到我们的数据后都能更方便的进行统一处理。

**​定义以下结果封装类**

com.sangeng.common.ResponseResult

```java
@JsonInclude(JsonInclude.Include.NON_NULL)//某个属性不为null，才会把它格式化转换成json
@Data
@NoArgsConstructor
public class ResponseResult<T {
    /**
     * 状态码
     */
    private Integer code;
    /**
     * 提示信息，如果有错误时，前端可以获取该字段进行提示
     */
    private String msg;
    /**
     * 查询到的结果数据，
     */
    private T data;

    public ResponseResult(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public ResponseResult(Integer code, T data) {
        this.code = code;
        this.data = data;
    }

    public ResponseResult(Integer code, String msg, T data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }
}
```

## 跨域请求

**什么是跨域**

​浏览器出于安全的考虑，使用XMLHttpRequest对象发起HTTP请求时必须遵守同源策略（协议，域名，端口号都一致才算同源），否则就是跨域的HTTP请求，默认情况下是被禁止的。

**CORS解决跨域**

​CORS是一个W3C标准，全称是”跨域资源共享”（Cross-origin resource sharing），允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制。

​它通过服务器增加一个特殊的Header[Access-Control-Allow-Origin]来告诉客户端跨域的限制，如果浏览器支持CORS、并且判断Origin通过的话，就会允许XMLHttpRequest发起跨域请求。

**SpringBoot使用CORS解决跨域的两种方法**

两种方法本质上没区别：都是把`WebMvcConfigurer`的实现类注入到容器中，并且重写`addCorsMappings`方法
实际开发中方法二更加常用

*方式一：使用@CrossOrigin*

在支持跨域的方法或者是Controller上加上`@CrossOrigin`注解

```java
@RestController
@RequestMapping("/user")
@CrossOrigin
public class UserController {

    @Autowired
    private UserServcie userServcie;

    @RequestMapping("/findAll")
    public ResponseResult findAll(){
        //调用service查询数据 ，进行返回
        List<User users = userServcie.findAll();

        return new ResponseResult(200,users);
    }
}
```

方法二：使用`WebMvcConfigurer`的`addCorsMappings`方法配置`CorsInterceptor`(这种更简便)

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
      // 设置允许跨域的路径
        registry.addMapping("/**")
                // 设置允许跨域请求的域名
                .allowedOriginPatterns("*")
                // 是否允许cookie
                .allowCredentials(true)
                // 设置允许的请求方式
                .allowedMethods("GET", "POST", "DELETE", "PUT")
                // 设置允许的header属性
                .allowedHeaders("*")
                // 跨域允许时间
                .maxAge(3600);
    }
}
```

**原理**：

## `@ControllerAdvice`统一异常处理（和SpringMVC一样）

这一部分和SpringMVC的一样，那里更详细

前后端分离的项目中，异常处理一般是把异常信息封装到Json中写入响应体。无论是哪种情况

**自定义一个异常处理控制器**

1. 这个类要添加`@ControllerAdvice`注解
2. 在类上加`@Component`注解注入容器，注意要保证类能被组件扫描到。
3. 处理异常的方法添加`@ExceptionHandler(异常类型.class)`注解，标识可以处理的异常

**例子（前后端分离的项目：把异常信息封装为对象，转换成json写入响应体）**

```java
@ControllerAdvice
public class MyControllerAdvice {

    @ExceptionHandler(RuntimeException.class)
    @ResponseBody
    public ResponseResult handlerException(Exception e){
        //获取异常信息，存入ResponseResult的msg属性
        String message = e.getMessage();
        ResponseResult result = new ResponseResult(300,message);//实际项目中300设为常量，易维护
        //把ResponseResult作为返回值返回，要求到时候转换成json存入响应体中
        return result;
    }
}
```

## 获取web原生对象（了解）

在web阶段经常要使用到request，response，session等对象。

通过SpringMVC也可以获取到这些对象。

*注：在MVC中很少获取这些对象了，因为有更简便的方式（获取参数，请求头，往响应体写数据等等操作，都有对应的注解方便开发）来避免使用这些原生对象相对繁琐的API*

**只需要在方法上添加对应类型的参数即可**

SpringMVC会把需要的对象传给形参

```java
@RestController
public class TestController {

    @RequestMapping("/getRequestAndResponse")
    public ResponseResult getRequestAndResponse(HttpServletRequest request, HttpServletResponse response, HttpSession session){
        System.out.println(request);
        return new ResponseResult(200,"成功");
    }
}
```

## 自定义参数解析

如果想实现像获取请求体中的数据那样(在Handler方法的参数上增加一个@RepuestBody注解就可以获取到对应的数据),就可以使用HandlerMethodArgumentResolver来实现自定义的参数解析。

①定义用来标识的注解

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface CurrentUserId {

}
```

②创建类实现HandlerMethodArgumentResolver接口并重写其中的方法

**注意加上@Component注解注入Spring容器**

```java
public class UserIdArgumentResolver implements HandlerMethodArgumentResolver {

    //判断方法参数使用能使用当前的参数解析器进行解析
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        //如果方法参数有加上CurrentUserId注解，就能把被我们的解析器解析
        return parameter.hasParameterAnnotation(CurrentUserId.class);
    }
    //进行参数解析的方法，可以在方法中获取对应的数据，然后把数据作为返回值返回。方法的返回值就会赋值给对应的方法参数
    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        //获取请求头中的token
        String token = webRequest.getHeader("token");
        if(StringUtils.hasText(token)){
            //解析token，获取userId
            Claims claims = JwtUtil.parseJWT(token);
            String userId = claims.getSubject();
            //返回结果
            return userId;
        }
        return null;
    }
}
```

③配置参数解析器

```java
@Configuration
public class ArgumentResolverConfig implements WebMvcConfigurer {

    @Autowired
    private UserIdArgumentResolver userIdArgumentResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver resolvers) {
        resolvers.add(userIdArgumentResolver);
    }
}
```

④测试

在需要获取UserId的方法中增加对应的方法参数然后使用@CurrentUserId进行标识即可获取到数据

```java
@RestController
@RequestMapping("/user")
//@CrossOrigin
public class UserController {

    @Autowired
    private UserServcie userServcie;

    @RequestMapping("/findAll")
    public ResponseResult findAll(@CurrentUserId String userId) throws Exception {
        System.out.println(userId);
        //调用service查询数据 ，进行返回s
        List<User users = userServcie.findAll();

        return new ResponseResult(200,users);
    }
}
```

## 开启声明式事务

直接在需要事务控制的方法上加上对应的注解`@Transactional`

```java
@Service
public class UserServiceImpl implements UserServcie {

    @Autowired
    private UserMapper userMapper;

    @Override
    public List<User findAll() {
        return userMapper.findAll();
    }

    @Override
    @Transactional
    public void insertUser() {
        //添加2个用户到数据库
        User user = new User(null,"sg666",15,"上海");
        User user2 = new User(null,"sg777",16,"北京");
        userMapper.insertUser(user);
        System.out.println(1/0);
        userMapper.insertUser(user2);
    }
}
```

## AOP

SpringBoot中默认是开启AOP功能的。如果不想开启AOP功能可以使用如下配置设置为false

```yml
spring:
  aop:
    auto: false
```

## 使用步骤

①添加依赖

```xml
        <dependency
            <groupIdorg.springframework.boot</groupId
            <artifactIdspring-boot-starter-aop</artifactId
        </dependency
```

②自定义注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface InvokeLog {
}
```

③定义切面类

```java
@Aspect  //标识这是一个切面类
@Component
public class InvokeLogAspect {

    //确定切点
    @Pointcut("@annotation(com.sangeng.aop.InvokeLog)")
    public void pt(){
    }

    @Around("pt()")
    public Object printInvokeLog(ProceedingJoinPoint joinPoint){
        //目标方法调用前
        Object proceed = null;
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        String methodName = signature.getMethod().getName();
        System.out.println(methodName+"即将被调用");
        try {
            proceed = joinPoint.proceed();
            //目标方法调用后
            System.out.println(methodName+"被调用完了");
        } catch (Throwable throwable) {
            throwable.printStackTrace();
            //目标方法出现异常了
            System.out.println(methodName+"出现了异常");
        }
        return proceed;
    }
}
```

④在需要正确的地方增加对应的注解

```Java
@Service
public class UserServiceImpl implements UserServcie {

    @Autowired
    private UserMapper userMapper;

    @Override
    @InvokeLog  //需要被增强方法需要加上对应的注解
    public List<User findAll() {
        return userMapper.findAll();
    }
}
```

## 切换动态代理

有的时候我们需要修改AOP的代理方式。

可以使用以下方式修改：

配置文件中配置spring.aop.proxy-target-class为false可以切换为使用jdk动态代理。该配置默认值为true，使用cglib动态代理。

```yml
spring:
  aop:
    proxy-target-class: false #切换为jdk动态代理的方式
```

也可以这么写：

```java
@SpringBootApplication
@EnableAspectJAutoProxy(proxyTargetClass = false)//修改代理方式
public class WebApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(WebApplication.class, args);
    }
}
```

# 日志系统

SpringBoot提供了丰富的日志系统，它几乎是开箱即用的。

## 开启日志

开启日志

```yml
debug: true #开启日志
logging:
  level:
    com.sangeng: debug #设置日志级别
```

## 花里胡哨

## 日志门面和日志实现

首先要区分一下，什么是日志门面（Facade）什么是日志实现。

之前学习的JUL实际上就是一种日志实现，可以直接使用JUL提供的日志框架来规范化打印日志

而日志门面，如Slf4j，是把不同的日志系统的实现进行了具体的抽象化，只提供了统一的日志使用接口，使用时只需要按照其提供的接口方法进行调用即可。由于它只是一个接口，并不是一个具体的可以直接单独使用的日志框架，所以最终日志的格式、记录级别、输出方式等都要通过接口绑定的具体的日志系统（log4j、logback、java.util.logging等）来实现

日志门面和日志实现就像JDBC和数据库驱动一样，一个是画大饼的，一个是真的去做饼的。

![img](https://upload-images.jianshu.io/upload_images/2909474-b5127a18b3eda3ec.png?imageMogr2/auto-orient/strip|imageView2/2/w/888)

现在有一个问题：不同的框架可能使用了不同的日志框架，这个时候出现众多日志框架并存的情况。

现在希望的是所有的框架一律使用日志门面（Slf4j）进行日志打印，该怎么去解决？（将其他框架依赖的日志框架直接更换为Slf4j是不现实的）

这时，可以采取类似于偷梁换柱的做法，只保留不同日志框架的接口和类定义等关键信息，而将实现全部定向为Slf4j调用。相当于有着和原有日志框架一样的外壳，对于其他框架来说依然可以使用对应的类进行操作，而具体如何执行，真正的内心已经是Slf4j的了。

![img](https://upload-images.jianshu.io/upload_images/2909474-512f5cca92e05e59.png?imageMogr2/auto-orient/strip|imageView2/2/w/928)

所以，SpringBoot为了统一日志框架的使用，做了这些事情：

* 直接将其他依赖以前的日志框架剔除
* 导入对应日志框架的Slf4j中间包
* 导入自己官方指定的日志实现，并作为Slf4j的日志实现层

## 在SpringBoot中打印日志信息

SpringBoot使用的是Slf4j作为日志门面，Logback（[Logback](http://logback.qos.ch/) 是log4j 框架的作者开发的新一代日志框架，它效率更高、能够适应诸多的运行环境，同时天然支持SLF4J）作为日志实现，对应的依赖为：

```xml
<dependency
    <groupIdorg.springframework.boot</groupId
    <artifactIdspring-boot-starter-logging</artifactId
</dependency
```

如果需要打印日志，可以像这样：

```java
@RequestMapping("/login")
public String login(){
    Logger logger = LoggerFactory.getLogger(MainController.class);
    logger.info("用户访问了一次登陆界面");
    return "login";
}
```

也可以使用Lombok的`@Slf4j`一个注解搞定

```java
@Slf4j
@Controller
public class MainController {

    @RequestMapping("/login")
    public String login(){
        log.info("用户访问了一次登陆界面");
        return "login";
    }
```

日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，SpringBoot默认只会打印INFO以上级别的信息。

## 配置Logback日志

Logback官网：https://logback.qos.ch

和JUL一样，Logback也能通过编写对应的配置文件实现定制化。SpringBoot推荐将配置文件名称命名为`logback-spring.xml`表示这是SpringBoot下Logback专用的配置，可以使用SpringBoot 的高级Profile功能，它的内容类似于这样：

```xml
<?xml version="1.0" encoding="UTF-8"?
<configuration
    <!-- 配置 --
</configuration
```

最外层由`configuration`包裹，一旦编写，那么就会替换默认的配置。所以如果内部什么都不写的话，那么会导致我们的SpringBoot项目没有配置任何日志输出方式，控制台也不会打印日志。

**在`logback-spring.xml`配置一个控制台日志打印**：

直接导入并使用SpringBoot预设好的日志格式：`org/springframework/boot/logging/logback/defaults.xml`，然后设置对应的`appender`即可：

```xml
<?xml version="1.0" encoding="UTF-8"?
<configuration
    <!--  导入其他配置文件，作为预设  --
    <include resource="org/springframework/boot/logging/logback/defaults.xml" /

    <!--  Appender作为日志打印器配置，这里命名随意  --
    <!--  ch.qos.logback.core.ConsoleAppender是专用于控制台的Appender  --
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender"
        <encoder
            <pattern${CONSOLE_LOG_PATTERN}</pattern
            <charset${CONSOLE_LOG_CHARSET}</charset
        </encoder
    </appender

    <!--  指定日志输出级别，以及启用的Appender，这里就使用了我们上面的ConsoleAppender  --
    <root level="INFO"
        <appender-ref ref="CONSOLE"/
    </root
</configuration
```

配置完成后，控制台已经可以正常打印日志信息了。

**开启文件打印**：

只需要再配置一个对应的Appender即可：

```xml
<!--  ch.qos.logback.core.rolling.RollingFileAppender用于文件日志记录，它支持滚动  --
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"
    <encoder
        <pattern${FILE_LOG_PATTERN}</pattern
        <charset${FILE_LOG_CHARSET}</charset
    </encoder
    <!--  自定义滚动策略，防止日志文件无限变大，也就是日志文件写到什么时候为止，重新创建一个新的日志文件开始写  --
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy"
        <!--  文件保存位置以及文件命名规则，这里用到了%d{yyyy-MM-dd}表示当前日期，%i表示这一天的第N个日志  --
        <FileNamePatternlog/%d{yyyy-MM-dd}-spring-%i.log</FileNamePattern
        <!--  到期自动清理日志文件  --
        <cleanHistoryOnStarttrue</cleanHistoryOnStart
        <!--  最大日志保留时间  --
        <maxHistory7</maxHistory
        <!--  最大单个日志文件大小  --
        <maxFileSize10MB</maxFileSize
    </rollingPolicy
</appender

<!--  指定日志输出级别，以及启用的Appender，这里就使用了我们上面的ConsoleAppender  --
<root level="INFO"
    <appender-ref ref="CONSOLE"/
    <appender-ref ref="FILE"/
</root
```

配置完成后，日志文件也能自动生成了。

我们也可以魔改官方提供的日志格式，官方文档：https://logback.qos.ch/manual/layouts.html

这里需要提及的是MDC机制，Logback内置的日志字段还是比较少，如果需要打印有关业务的更多的内容，包括自定义的一些数据，需要借助logback MDC机制，MDC为“Mapped Diagnostic Context”（映射诊断上下文），即将一些运行时的上下文数据通过logback打印出来；此时需要借助org.sl4j.MDC类。

比如现在需要记录是哪个用户访问我们网站的日志，只要是此用户访问我们网站，都会在日志中携带该用户的ID，我们希望每条日志中都携带这样一段信息文本，而官方提供的字段无法实现此功能，这时就需要使用MDC机制：

```java
@Slf4j
@Controller
public class MainController {

    @RequestMapping("/login")
    public String login(){
      	//这里就用Session代替ID吧
        MDC.put("reqId", request.getSession().getId());
        log.info("用户访问了一次登陆界面");
        return "login";
    }
```

通过这种方式，就可以向日志中传入自定义参数了，日志中添加这样一个占位符`%X{键值}`，名字保持一致：

```xml
 %clr([%X{reqId}]){faint} 
```

这样当我们向MDC中添加信息后，只要是当前线程（本质是ThreadLocal实现）下输出的日志，都会自动替换占位符。

## 自定义Banner

Banner部分和日志部分是独立的，SpringBoot启动后，会先打印Banner部分，这个Banner部分也是可以自定义的。

直接在配置文件所在目录下创建一个名为`banner.txt`的文本文档，内容随便

```txt
//                          _ooOoo_                               //
//                         o8888888o                              //
//                         88" . "88                              //
//                         (| ^_^ |)                              //
//                         O\  =  /O                              //
//                      ____/`---'\____                           //
//                    .'  \\|     |//  `.                         //
//                   /  \\|||  :  |||//  \                        //
//                  /  _||||| -:- |||||-  \                       //
//                  |   | \\\  -  /// |   |                       //
//                  | \_|  ''\---/''  |   |                       //
//                  \  .-\__  `-`  ___/-. /                       //
//                ___`. .'  /--.--\  `. . ___                     //
//              ."" '<  `.___\_<|_/___.'  '"".                  //
//            | | :  `- \`.;`\ _ /`;.`/ - ` : | |                 //
//            \  \ `-.   \_ __\ /__ _/   .-` /  /                 //
//      ========`-.____`-.___\_____/___.-`____.-'========         //
//                           `=---='                              //
//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^        //
//             佛祖保佑          永无BUG         永不修改             //
```

可以使用在线生成网站进行生成自己的个性Banner：https://www.bootschool.net/ascii

甚至还可以使用颜色代码来为文本切换颜色：

```
${AnsiColor.BRIGHT_GREEN}  //绿色
```

也可以获取一些常用的变量信息：

```
${AnsiColor.YELLOW} 当前 Spring Boot 版本：${spring-boot.version}
```

# 多环境配置

在日常开发中，项目会有多个环境。例如开发环境（develop）也就是研发过程中疯狂敲代码修BUG阶段，生产环境（production ）就是项目开发得差不多了，可以放在服务器上跑了。

不同的环境下，可能配置文件也存在不同（端口、上下文路径、数据库配置等等），但是又不可能切换环境的时候又去重新写一次配置文件（麻烦）。

**为什么用profile**

profile的出现就是为了解决这个问题。可以把不同环境下不同的配置放到对应的profile文件中，然后把不同环境下都相同的配置放到application.yml文件中。

**如何进行自由切换**

分别创建两个环境的profile配置文件：`application-dev.yml`和`application-prod.yml`，分别表示开发环境和生产环境的配置文件。


比如开发环境使用的服务器端口为8080，而生产环境下可能就需要设置为80或是443端口，那么这个时候就需要不同环境下的配置文件进行区分：

```yaml
server:
  port: 8080
```

```yaml
server:
  port: 80
```

然后在`application.yml`配置文件指定目前使用的是`application-dev.yml`

```yaml
spring:
  profiles:
    active: dev
```

也可以使用虚拟机参数来指定激活环境。例如：-Dspring.profiles.active=dev

​也可以使用命令行参数来激活环境。例如：--spring.profiles.active =dev

**SpringBoot自带的Logback日志系统也是支持多环境配置的**

比如现在希望开发环境（dev）下输出日志到控制台，而生产环境（prod）下只需要输出到文件即可，这时就需要进行环境配置：

在`logback-spring.xml`添加以下内容(*注意：`springProfile`是区分大小写的！*)

```xml
<springProfile name="dev"
    <root level="INFO"
        <appender-ref ref="CONSOLE"/
        <appender-ref ref="FILE"/
    </root
</springProfile

<springProfile name="prod"
    <root level="INFO"
        <appender-ref ref="FILE"/
    </root
</springProfile
```

**打包问题**

目前虽然可以切换开发环境，但是打包的时候依然是所有配置文件全部打包，如果希望生产环境中不要打包开发环境下的配置文件，就只能找Maven解决

修改`pom.xml`：

先添加环境

```xml
<!--分别设置开发，生产环境--
<profiles
    <!-- 开发环境 --
    <profile
        <iddev</id
        <activation
            <activeByDefaulttrue</activeByDefault
        </activation
        <properties
            <environmentdev</environment
        </properties
    </profile
    <!-- 生产环境 --
    <profile
        <idprod</id
        <activation
            <activeByDefaultfalse</activeByDefault
        </activation
        <properties
            <environmentprod</environment
        </properties
    </profile
</profiles
```

接着，根据环境的不同，排除其他环境的配置文件：

```xml
<resources
<!--排除配置文件--
    <resource
        <directorysrc/main/resources</directory
        <!--先排除所有的配置文件--
        <excludes
            <!--使用通配符，当然可以定义多个exclude标签进行排除--
            <excludeapplication*.yml</exclude
        </excludes
    </resource

    <!--根据激活条件引入打包所需的配置和文件--
    <resource
        <directorysrc/main/resources</directory
        <!--引入所需环境的配置文件--
        <filteringtrue</filtering
        <includes
            <includeapplication.yml</include
            <!--根据maven选择环境导入配置文件--
            <includeapplication-${environment}.yml</include
        </includes
    </resource
</resources
```

最后将Maven中的`environment`属性，传递给SpringBoot的配置文件`application.yml`，在构建时替换为对应的值：

```yaml
spring:
  profiles:
    active: '@environment@'  #注意YAML配置文件需要加单引号，否则会报错
```

这样，根据Maven环境的切换，SpringBoot的配置文件也会进行对应的切换。

打开Maven栏目，直接勾选即可实现开发环境的自由切换了。

*注意切换环境之后要重新加载一下Maven项目，不然不会生效！*

# 邮件功能

## 邮件Mail

很多的网站中，都会遇到邮件或是手机号验证，也就是通过你的邮箱或是手机短信去接受网站发给你的注册验证信息，填写验证码之后，就可以完成注册了，同时，网站也会绑定你的手机号或是邮箱。

像这样的功能，SpringBoot已经提供了封装好的邮件模块使用：

```xml
<dependency
    <groupIdorg.springframework.boot</groupId
    <artifactIdspring-boot-starter-mail</artifactId
</dependency
```

## 了解电子邮件

电子邮件是一种通信方式，是互联网应用最广的服务。通过网络的电子邮件系统，用户可以以非常低廉的价格（不管发送到哪里，都只需负担网费，实际上就是把信息发送到对方服务器而已）、非常快速的方式，与世界上任何一个地方的电子邮箱用户联系。

要在Internet上提供电子邮件功能，必须有专门的电子邮件服务器。例如现在Internet很多提供邮件服务的厂商：新浪、搜狐、163、QQ邮箱等，都有自己的邮件服务器。这些服务器类似于现实生活中的邮局，主要负责接收用户投递过来的邮件，并把邮件投递到邮件接收者的电子邮箱中。

所有的用户都可以在电子邮件服务器上申请一个账号用于邮件发送和接收

和Http一样，邮件发送也有自己的协议，也就是约定邮件数据长啥样以及如何通信。

![img](https://images2015.cnblogs.com/blog/851491/201612/851491-20161202143243756-1715308358.png)

比较常用的协议有两种：

1. SMTP协议（主要用于发送邮件 Simple Mail Transfer Protocol）
2. POP3协议（主要用于接收邮件 Post Office Protocol 3）

整个发送/接收流程大致如下：

![img](https://img2.baidu.com/it/u=3675146129,445744702&fm=253&fmt=auto&app=138&f=JPG?w=812&h=309)

每个邮箱服务器都有一个smtp发送服务器和pop3接收服务器，比如要从QQ邮箱发送邮件到163邮箱，那么只需要通过QQ邮箱客户端告知QQ邮箱的smtp服务器需要发送邮件，以及邮件的相关信息，然后QQ邮箱的smtp服务器就会将邮件发送到163邮箱的pop3服务器上，163邮箱会通过163邮箱客户端告知对应用户收到一封新邮件。

## 邮箱发送

想要实现给别人发送邮件，那么就需要连接到对应电子邮箱的smtp服务器上，并告知它要发送邮件。

SpringBoot已经将最基本的底层通信全部实现了，使用时只需要关心smtp服务器的地址以及要发送的邮件长啥样即可。

**以163邮箱为例**

① 首先去163邮箱设置中开启smtp/pop3服务，开启后会得到一个随机生成的密钥，这个就是密码

② 在配置文件中告诉SpringBootMail我们的smtp服务器的地址以及邮箱账号和密码

```yaml
spring:
  mail:
  	# 163邮箱的地址为smtp.163.com，直接填写即可
    host: smtp.163.com
    # 我的163邮箱账号
    username: 123456@163.com
    # 注意密钥是在开启smtp/pop3时自动生成的，记得保存一下，不然就找不到了
    password: AZJTOAWZESLMHTNI
    port: 465
    properties:
     from: m19925651773@163.com
     mail:
       smtp:
         socketFactory:
           class: javax.net.ssl.SSLSocketFactory
```

③ SimpleMailMessage封装邮件内容，JavaMailSender将邮件发送

```java
@SpringBootTest
class SpringBootTestApplicationTests {

    //JavaMailSender是专门用于发送邮件的对象，自动配置类已经提供了Bean
    @Autowired
    JavaMailSender sender;

    @Test
    void contextLoads() {
      	//SimpleMailMessage是一个比较简易的邮件封装，支持设置一些比较简单内容
        SimpleMailMessage message = new SimpleMailMessage();
      	//设置邮件标题
        message.setSubject("【XXX大学教务处】关于近期学校对您的处分决定");
      	//设置邮件内容
        message.setText("XXX同学您好，经监控和教务巡查发现，您近期存在旷课、迟到、早退、上课刷抖音行为，" +
                "现已通知相关辅导员，请手写5000字书面检讨，并在2023年4月27日17点前交到辅导员办公室。");
      	//设置邮件发送给谁，可以多个，这里就发给自己的QQ邮箱
        message.setTo("自己的qq号@qq.com");
      	//邮件发送者，这里要与配置文件中的保持一致
        message.setFrom("123456@163.com");
      	//发送
        sender.send(message);
    }

}
```

④ 如果需要添加附件等更多功能，可以使用MimeMessageHelper来完成：

```java
@Test
void contextLoads() throws MessagingException {
    //创建一个MimeMessage
    MimeMessage message = sender.createMimeMessage();
    //使用MimeMessageHelper来修改MimeMessage中的信息
    MimeMessageHelper helper = new MimeMessageHelper(message, true);
    helper.setSubject("Test");
    helper.setText("lbwnb");
    helper.setTo("你的QQ号@qq.com");
    helper.setFrom("javastudy111@163.com");
  	//发送修改好的MimeMessage
    sender.send(message);
}
```

## 邮件注册

邮箱注册流程：
1. 请求验证码
2. 生成验证码并发送给对应邮箱（临时有效，注意设定过期时间）
3. 用户输入验证码并填写注册信息
4. 验证通过注册成功

# 接口规则校验

用户发送的数据可能存在一些问题

比如在下面这个接口中，在正常情况下，用户名长度规定不小于5，如果用户发送的数据是没有问题的，那么就可以正常运行。但是如果用户发送的数据并不是按照规定的，那么就会直接报错
```java
@ResponseBody
@PostMapping("/submit")
public String submit(String username,
                     String password){
    System.out.println(username.substring(3));
    System.out.println(password.substring(2, 10));
    return "请求成功!";
}
```

可以在请求来后进行校验判断：

```java
@ResponseBody
@PostMapping("/submit")
public String submit(String username,
                     String password){
    if(username.length() > 3 || password.length() > 10) {
        System.out.println(username.substring(3));
        System.out.println(password.substring(2, 10));
        return "请求成功!";
    } else {
        return "请求失败";
    }
}
```

但是如果每一个接口都这样去进行配置，太麻烦了。

***
这里可以使用SpringBoot提供了的接口校验框架：

**导入依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

**使用注解完成接口的校验**

```java
@Slf4j
@Validated//在Controller上开启接口校验
@Controller
public class TestController {

    ...

    @ResponseBody
    @PostMapping("/submit")
    public String submit(@Length(min = 3) String username,  //使用@Length注解一步到位
                         @Length(min = 10) String password){
        System.out.println(username.substring(3));
        System.out.println(password.substring(2, 10));
        return "请求成功!";
    }
}
```

依然会抛出异常，**配置异常处理Controller**来自行处理这类异常：

```java
@ControllerAdvice
public class ValidationController {

    @ResponseBody
    @ExceptionHandler(ConstraintViolationException.class)
    public String error(ValidationException e){
        return e.getMessage();   //出现异常直接返回消息
    }
}
```

***

如果接口接收的**参数是对象形式的**，验证方法如下：

```java
@ResponseBody
@PostMapping("/submit")  //在参数上添加@Valid注解表示需要验证
public String submit(@Valid Account account){
    System.out.println(account.getUsername().substring(3));
    System.out.println(account.getPassword().substring(2, 10));
    return "请求成功!";
}
```

```java
@Data
public class Account {
    //在对应的字段上添加校验的注解
    @Length(min = 3) 
    String username;
    @Length(min = 10)
    String password;
}
```

**修改下异常处理Controller**
对于实体类接收参数的验证，会抛出MethodArgumentNotValidException异常

```java
@ResponseBody
@ExceptionHandler({ConstraintViolationException.class, MethodArgumentNotValidException.class})
public String error(Exception e){
    if(e instanceof ConstraintViolationException exception) {
        return exception.getMessage();
    } else if(e instanceof MethodArgumentNotValidException exception){
        if (exception.getFieldError() == null) return "未知错误";
        return exception.getFieldError().getDefaultMessage();
    }
    return "未知错误";
}
```

***

**更多验证注解**

|   验证注解   |                        验证的数据类型                        |                           说明                           |
| :----------: | :----------------------------------------------------------: | :------------------------------------------------------: |
| @AssertFalse |                       Boolean,boolean                        |                      值必须是false                       |
| @AssertTrue  |                       Boolean,boolean                        |                       值必须是true                       |
|   @NotNull   |                           任意类型                           |                       值不能是null                       |
|    @Null     |                           任意类型                           |                       值必须是null                       |
|     @Min     | BigDecimal、BigInteger、byte、short、int、long、double 以及任何Number或CharSequence子类型 |                   大于等于@Min指定的值                   |
|     @Max     |                             同上                             |                   小于等于@Max指定的值                   |
| @DecimalMin  |                             同上                             |         大于等于@DecimalMin指定的值（超高精度）          |
| @DecimalMax  |                             同上                             |         小于等于@DecimalMax指定的值（超高精度）          |
|   @Digits    |                             同上                             |                限制整数位数和小数位数上限                |
|    @Size     |               字符串、Collection、Map、数组等                |       长度在指定区间之内，如字符串长度、集合大小等       |
|    @Past     |       如 java.util.Date, java.util.Calendar 等日期类型       |                    值必须比当前时间早                    |
|   @Future    |                             同上                             |                    值必须比当前时间晚                    |
|  @NotBlank   |                     CharSequence及其子类                     |         值不为空，在比较时会去除字符串的首位空格         |
|   @Length    |                     CharSequence及其子类                     |                  字符串长度在指定区间内                  |
|  @NotEmpty   |         CharSequence及其子类、Collection、Map、数组          | 值不为null且长度不为空（字符串长度不为0，集合大小不为0） |
|    @Range    | BigDecimal、BigInteger、CharSequence、byte、short、int、long 以及原子类型和包装类型 |                      值在指定区间内                      |
|    @Email    |                     CharSequence及其子类                     |                     值必须是邮件格式                     |
|   @Pattern   |                     CharSequence及其子类                     |               值需要与指定的正则表达式匹配               |
|    @Valid    |                        任何非原子类型                        |                     用于验证对象属性                     |



# Swagger接口管理

前后端分离的开发模式中，前端和后端的工作由不同的工程师完成。在这种开发模式下，维持一份及时更新且完整的Rest API 文档将会极大的提高工作效率。

传统意义上的文档都是后端开发人员手动编写的。这种方式很难保证文档的及时性，久而久之也就会失去其参考意义，反而还会加大沟通成本。

而Swagger提供了一个全新的维护API文档的方式

1. 代码变，文档变。只需要少量的注解，Swagger 就可以根据代码自动生成 API文档，很好的保证了文档的时效性。
2. 跨语言性，支持 40 多种语言。
3. Swagger UI呈现出来的是一份可交互式的API文档。我们可以直接在文档页面尝试 API 的调用，省去了准备复杂的调用参数的过程。

***

**先导入依赖**
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.1.0</version>
</dependency>
```

项目启动之后，直接访问：http://localhost:8080/swagger-ui/index.html， 就能看到开发文档
开发文档中自动包含了项目种定义的接口，并且还有对应的实体类，不仅仅可以展示接口，也可以直接在上面进行调试

**配置整体页面信息，展示自定义的文本信息**

```java
@Bean
public OpenAPI springDocOpenAPI() {
        return new OpenAPI().info(new Info()
                        .title("图书管理系统 - 在线API接口文档")   //设置API文档网站标题
                        .description("这是一个图书管理系统的后端API文档，欢迎前端人员查阅！") //网站介绍
                        .version("2.0")   //当前API版本
                        .license(new License().name("啦啦啦")  //遵循的协议，这里拿来写其他的也行
                                .url("https://www.baidu.com")));
}
```

**为Controller编写API描述信息**

在类名称上面添加`@Tag`注解，并填写相关信息，来为当前的Controller设置描述信息。

```java
//使用@Tag注解来添加Controller描述信息
@Tag(name = "账户验证相关", description = "包括用户登录、注册、验证码请求等操作。")
public class TestController {
	...
}
```

**为请求映射配置描述信息**

```java
@ApiResponses({
       //不同返回状态码描述
       @ApiResponse(responseCode = "200", description = "测试成功"),
       @ApiResponse(responseCode = "500", description = "测试失败")   
})
@Operation(summary = "请求用户数据测试接口")   //接口功能描述
@ResponseBody
@GetMapping("/hello")
//请求参数描述和样例
public String hello(@ApiParam(description = "测试文本数据", example = "KFCvivo50") String text) {
    return "Hello World";
}
```

**不需要展示在文档中的接口，也可以将其忽略掉**

```java
@Hidden
@ResponseBody
@GetMapping("/hello")
public String hello() {
    return "Hello World";
}
```

**为实体类编写对应的API接口文档**

```java
@Data
@Schema(description = "用户信息实体类")
public class User {
    @Schema(description = "用户编号")
    int id;
    @Schema(description = "用户名称")
    String name;
    @Schema(description = "用户邮箱")
    String email;
    @Schema(description = "用户密码")
    String password;
}
```
***

接口文档只适合在开发环境下生成，如果是生产环境，需要关闭文档：

```java
springdoc:
  api-docs:
    enabled: false
```

# 指标监控

​日常开发中需要对程序内部的运行情况进行监控，比如：健康度、运行指标、日志信息、线程状况等等。而SpringBoot的监控Actuator就可以解决这些问题。

**常用端点**

| 端点名称         | 描述                                      |
| :--------------- | :---------------------------------------- |
| `beans`          | 显示应用程序中所有Spring Bean的完整列表。 |
| `health`         | 显示应用程序运行状况信息。                |
| `info`           | 显示应用程序信息。                        |
| `loggers`        | 显示和修改应用程序中日志的配置。          |
| `metrics`        | 显示当前应用程序的“指标”信息。            |
| `mappings`       | 显示所有`@RequestMapping`路径列表。       |
| `scheduledtasks` | 显示应用程序中的计划任务。                |

## 直接访问Actuator（了解就行）

直接访问actuator只是响应个Json数据，阅读困难（特别是很复杂的时候）

1. 添加依赖

```xml
<dependency
 	<groupIdorg.springframework.boot</groupId
 	<artifactIdspring-boot-starter-actuator</artifactId
</dependency
```

2. 访问监控接口

http://localhost:81/actuator

3. 配置启用监控端点

```yml
management:
  endpoints:
    enabled-by-default: true #配置启用所有端点
	web:
      exposure:
        include: "*" #web端暴露所有端点
  endpoint:
  health:
      show-details: always #展示所有的健康信息
```

## 图形化界面SpringBoot Admin

这个东西就是把actuator响应的Json数据进行解析，展示在图形化界面上

Admin Server是用来接收监控信息，并进行展示的服务

Admin client提供监控信息（比如说要监控的一个项目模块就是一个Admin client）

配置启用监控端点

```yml
management:
  endpoints:
    enabled-by-default: true #配置启用所有端点
	web:
      exposure:
        include: "*" #web端暴露所有端点
  endpoint:
  health:
      show-details: always #展示所有的健康信息
```


**先创建SpringBoot Admin Server应用**

引入spring-boot-admin-starter-server依赖

```xml
<dependency
    <groupIdde.codecentric</groupId
    <artifactIdspring-boot-admin-starter-server</artifactId
</dependency
```

然后在启动类上加上@EnableAdminServer注解

**然后配置SpringBoot Admin client应用**

在需要监控的模块中加上spring-boot-admin-starter-client依赖

```xml
<dependency
    <groupIdde.codecentric</groupId
    <artifactIdspring-boot-admin-starter-client</artifactId
    <version2.3.1</version
</dependency
```

然后配置SpringBoot Admin Server的地址

```yml
spring:
  boot:
    admin:
      client:
        url: http://localhost:8888 #配置 Admin Server的地址
```

# 打包运行

Maven生命周期中的`package`会自动将项目打包为可直接运行的Jar包

打包的过程中还会完整的将项目跑一遍进行测试，如果不想测试直接打包，可以手动使用以下命令：

```shell
mvn package  -DskipTests
```

打包后，会直接得到一个名为`springboot-study-0.0.1-SNAPSHOT.jar`的文件，在它的所在位置CMD窗口中输入命令就可以运行项目了：

```shell
java -jar springboot-study-0.0.1-SNAPSHOT.jar
```

也可以创建start.bat文件，内容就是上述命令，那么双击这个文件就可以运行项目
