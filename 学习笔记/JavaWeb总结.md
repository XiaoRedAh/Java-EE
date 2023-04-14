# 浅学JavaWeb小结
## 前置
### 理解DAO层，UI层，Business logic层
> DAO(data access object) should perform raw database operations and translate them to some higher level constructs (objects, collections).  

> Services should call DAOs and perform business operations. Typically transactions demarcation is performed on service layer to span several DAO calls.

>Finally DAO should abstract business logic from persistence details, ideally allowing to switch persistence layer without business logic (services) changes. This is hardly ever possible due to leaking abstraction of persistence providers (e.g. lazy loading).
### 理解servlet和web service
>A web service is a service that provides service methods to its clients using either the REST programming paradigm or the SOAP protocol for communication.

>A servlet is a Java-specific way of writing software that responds to HTTP requests (transport HTTP requests and responses).It can be used to write a web application with JSPs and HTML, or to serve XML and JSON responses (as in a RESTful service) and of course also to receive and return SOAP messages.

>A more comprehensive and practical approach is to write a web service with a framework and to publish it on an application server or servlet container such as Tomcat or JBoss. In this case you would use a Servlet to handle the transport of the HTTP requests which transmit your SOAP or REST messages.
### Tomcat只是servlet容器
>A servlet-container supports only the servlet API (including JSP, JSTL).
So it is a Web Server that must be able to run Java Servlets.It is possible to run most of the JavaEE technologies on a servlet-container, but you have to install a standalone implementation of the particular technology.

>Web Server, on the other hand, is a broader term including all software running and communicating through HTTP protocol.An application server supports the whole JavaEE - EJB, JMS, CDI, JTA, the servlet API (including JSP, JSTL), etc.

>All Servlet Containers are Web Servers.
Not all Web Servers are Servlet Containers. I.E. Apache HTTP Server can not run Java Servlet directly.



### Mysql+Mybatis使用
1. 导入mysql，mybatis，lombox依赖

2. 连接数据库

3. 创建并编写mybatis-config.xml，创建对应实体类的映射接口，创建并编写对应的映射配置文件xml，并将其添加到mybatis-config.xml中

4. 可选：创建SqlUtil来管理SqlSessionFactory的创建

5. 在要用数据库的地方，用一个SqlSession对象的getMapper函数创建映射接口类对象a,就可用a来进行定义好的数据库操作了
### Servlet
1. 创建继承于HttpServlet的类，`@WebServlet(路径)`给创建的servlet注册一个访问路径
2. html的表单添加method属性(提交该表单执行`get`还是`post`请求)，
action属性（响应该表单的servlet是哪个）
3. 在对应的servlet中实现`doGet()`或`doPost()`函数
前后端交互：前端发送get，post等请求，后端要有对应的doPost，doGet进行相应
### 上传，下载文件
* 前端从后端下载文件：
输入流读到文件后将它复制给输出流，从后端输出到前端
* 前端上传文件给后端：
输入流读到上传的文件后将其复制给输出流，输出到指定路径上
### 重定向和请求转发 
重定向代码(转到`@WebServlet("\time")`)
```java
resp.sendRedirect("time");
```
请求转发代码(转到`@WebServlet("\time")`)
```java
req.getRequestDispatcher("/time").forward(req, resp);
```
两者的区别为：
* 请求转发是一次请求，重定向是两次请求
* 请求转发地址栏不会发生改变， 重定向地址栏会发生改变
* 请求转发可以共享请求参数 ，重定向之后，就获取不了共享参数了
* 请求转发只能转发给内部的Servlet
### ServletContext

### Cookie
> 帮客户端(浏览器)保存一些数据，辅助服务端操作，即服务端从客户端获取数据（比如：
将正确的用户名，密码保存到Cookie后接下来的maxAge周期内都可免密登录，不用到登录界面）
1. 创建Cookie可有多个，用resp的`addCookie()`函数添加创建的对象
   可用`setMaxAge()`规定 Cookie失效的时间
maxAge默认-1：关闭浏览器，cookie才会被删除，不关则一直在
            0：删除
            正数：cookie在浏览器上保存多少秒，保存期过后就被删除（期间关闭浏览器，
cookie也会保存）
2. 获得客户端req传来的Cookie（因为有多个，所以用数组）：`req.getCookies（）`
  
### Session
服务端为每个浏览器（会话）创建一个Session，这样后端就可以知道是哪个用户在发送请求。用户访问浏览器的数据就保存在里面，需
要用到时，可以拿出来
### 过滤器
1. 创建过滤器类，继承HttpFilter，`@WebFilter（路径）`来确定访问哪里时要用到该过滤器
2. 重写`doFilter()`函数，里面写过滤时的操作。因为过滤器可以有多个，因此在结尾要用到`chain.getFilter(req,res)`让下一个过滤器工作【实际上时递归调用，看笔记】
## 开发一个javaweb项目
**涉及：
MySql+Mybatis
Lombox
Servlet
Thymeleaf
Maven
Tomcat**
### 前置工作
1. Maven框架+Servlet依赖+Thymeleaf依赖（JavaEE 8里有）+Tomcat服务器  
若是javaEE 8，创建后pom.xml里的Servlet依赖要改：
```java
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>5.0.0</version>
    <scope>provided</scope>
</dependency>
```
1. 为项目创建好主要的软件包
   1. dao：放Mybatis实体映射对象的接口类
   2. entity：放项目需要用到的实体类
   3. filter：放项目需要用到的过滤器类
   4. service：放项目业务逻辑层方面的类  
   规范：先写接口，然后再开个impl包，在包里创Java类去实现定义的接口
   1. servlet：放创建的Servlet容器
   2. utils：将Mybatis和Thymeleaf写成工具类，方便使用

2. 静态资源放webapp下，动态资源放在resources
3. 为项目做前置工作：导入依赖，编写需要的配置文件，连接数据库
4. 根据需求完成类的实现（后端代码修改后，前端代码也要有对应修改）
### 可复用的代码
#### 导入依赖(mysql,mybatis,lombox)
```xml
<dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.27</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.7</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.22</version>
            <scope>provided</scope>
        </dependency>
```
#### mybatis-config.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```
#### MyBatisUtil
```java
public class MybatisUtil {
    private static SqlSessionFactory factory;
    static{
        try {
            factory=new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
    public static SqlSession getSession(boolean autoCommit){
        return factory.openSession(autoCommit);
    }
}
```
#### ThymyleafUtil
```java
public class ThymeleafUtil {
    private static final TemplateEngine engine;
    static  {
        engine = new TemplateEngine();
        ClassLoaderTemplateResolver r = new ClassLoaderTemplateResolver();
        r.setCharacterEncoding("UTF-8");
        engine.setTemplateResolver(r);
    }

    public static void process(String template, IContext context, Writer writer){
        engine.process(template,context,writer);
    }
}
```
#### 利用Thymeleaf实现前后端交互(展示其主要用法)
**实例1：将后端数据展示在前端下拉框中**
* 后端
```java
 protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Context context=new Context();
        User user= (User) req.getSession().getAttribute("user");
        context.setVariable("usernameForheaderHtml",user.getUsername());//将登录的用户名显示在页面上（看header.html）
        context.setVariable("student_list",studentService.returnStudentList());//将学生信息展示到前端
        context.setVariable("activeBook_list",bookService.returnActiveBook());//将未借阅的书籍展示到前端
        ThymeleafUtil.process("addBorrow.html",context, resp.getWriter());
    }
```
* 前端页面
```html
<select class="select2 form-control" id="city" name="student">
<option th:each="students: ${student_list}" th:text="${students.getName()}" th:value="${students.getSid()}"></option>
</select>
```
**实例2：将后端的对象数组展示到前端表格+还书按钮功能实现**
* 后端
```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Context context=new Context();
        User user= (User) req.getSession().getAttribute("user");
        context.setVariable("usernameForheaderHtml",user.getUsername());//将登录的用户名显示在页面上（看header.html）
        context.setVariable("borrow_list",service.returnBorrowList());//从数据库获取借阅表，通过Thymeleaf和index.html展示到主页面
        ThymeleafUtil.process("index.html",context,resp.getWriter());
    }
```
* 前端
```html
<tbody>
  <tr th:each="borrow: ${borrow_list}">
      <td th:text="'#'+${borrow.getBookId()}"></td>
      <td th:text="${borrow.getBookName()}"></td>
      <td th:text="${borrow.getBorrowTime()}"></td>
      <td th:text="${borrow.getBorrowStudent()}"></td>
      <td th:text="'#'+${borrow.getStudentId()}"></td>
      <td class="relative">
          <a class="action-btn " th:href="'return-book?id='+${borrow.getId()}+'&bid='+${borrow.getBookId()}">
          </a>
      </td>
  </tr>
</tbody>
```
### 如何实现一个具体功能(一个功能对应一个Servlet)
#### 思考和代码编写顺序：  
Dao层(Mapper)—>业务逻辑层(Service)—>表现层(Servlet)—>前端代码   
* Dao层一般是用来执行Sql语句并映射到实体类
* Service层一般是使用Dao层的方法去完成特定的业务功能(比如下面的登录功能) 
* Servlet层是获取前端请求，调用对应的Service方法完成请求，并回复给前端相应的信息
* 前面的三层可能会要求前端代码的动态更新，因此要相应地修改前端代码(用Thymeleaf)  
    
#### 以登录功能为例  
涉及到的知识：过滤+数据库搜索+实体类映射+HttpSession记录登录成功
1. 过滤器工作  
除了访问静态资源或登录功能的Servlet，否则都要判断有没有登录，没有登录就重定向到登录功能的Servlet
2. 创建User实体类
3. Dao层实体类映射工作  
   创UserMapper接口类，在里面用注解方法声明明并实现一个`getUser()`
4. 业务逻辑层工作（service）  
创UserService接口类，在里面声明判断是否登录成功的函数声明`isLogin()`
创对应的UserServiceImpl实现类去实现接口
5. 表示层工作（Servlet）  
类里定义UserService的成员变量
`Init()`里创建UserService变量
`doGet()`里用Thymeleaf加载前端页面
`doPost()`里完成登录校验流程（判断登录是否成功用业务逻辑层的函数），`doPost()`管理登录成功后的操作和登录失败后的操作
### 打包发布
maven生命周期package打包生成war文件，将war文件放在tomcat服务器的webapps目录下，启动tomcat服务器即可在浏览器访问写好的网站  
localhost:8080/war文件的文件名

