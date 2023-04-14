# SSM开发流程
以图书馆里系统为例
使用spring5+maven+javaEE8+jdk1.8+Tomcat9
## 建数据表

**用户表users**

创建四列
* id：主键，自动递增
* name：姓名，设置为`UNIQUE`
* password：密码
* role：用户角色，用于授权

```sql
create table users
(
    id       int          not null,
    name     varchar(255) not null,
    password varchar(255) not null,
    role     varchar(255) not null,
    constraint ss_pk
        primary key (id)
);
```

```sql
create unique index unique_name
    on users (name);
```

**学生表student**

创建五列
* sid：主键，自动递增
* uid：对应于user表的id。设置外键：与users表的id绑定
* name：姓名
* sex：性别。枚举类型，默认'男'
* grade:年级

```sql
create table student
(
    sid   int                            not null,
    uid   int                            not null,
    name  varchar(255)                   not null,
    sex   enum ('男', '女') default '男' not null,
    grade varchar(255)                   not null,
    constraint student_pk
        primary key (sid),
    constraint f_uid
        foreign key (uid) references users (id)
);
```

**图书表book**

创建四列
* bid：主键
* title：书名    
* `desc`：书籍简介，因为是关键词，所以有``
* price：书籍价格

注意：和借阅表borrow有触发器，在借阅表里的书不能删除

```sql
create table book
(
    bid        int auto_increment
        primary key,
    title      varchar(20)   not null,
    `desc`     varchar(20)   not null,
    price      double        not null,
);
```

**借阅表borrow**

创建四列
* id：主键
* bid：被借阅的书的bid  
* sid：借阅人的id
* time：借阅时间

```sql
create table borrow
(
    id   int auto_increment
        primary key,
    bid  int      null,
    sid  int      null,
    time datetime null
);
```


## 搭建环境和前期工作

**导入依赖**

```xml
<!-- 建议为各个依赖进行分类，到后期我们的项目可能会导入很多依赖，添加注释会大幅度提高阅读效率 -->
<dependencies>
    <!--  Spring框架依赖  -->
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-web</artifactId>
        <version>5.5.3</version>
    </dependency>
  	<dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-config</artifactId>
        <version>5.5.3</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.14</version>
    </dependency>
    <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf-spring5</artifactId>
            <version>3.0.12.RELEASE</version>
        </dependency>
    <dependency>
            <groupId>org.thymeleaf.extras</groupId>
            <artifactId>thymeleaf-extras-springsecurity5</artifactId>
            <version>3.0.4.RELEASE</version>
    </dependency>

    <!--  持久层框架依赖  -->
		<dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.27</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.6</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.7</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.3.14</version>
        </dependency>
        <dependency>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
            <version>3.4.5</version>
        </dependency>

    <!--  其他工具框架依赖：Lombok、Slf4j  -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.22</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-jdk14</artifactId>
        <version>1.7.32</version>
    </dependency>

    <!--  ServletAPI  -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.1</version>
        <scope>provided</scope>
    </dependency>

    <!--  JUnit依赖  -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>${junit.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-engine</artifactId>
        <version>${junit.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

由于添加了Thymeleaf的SpringSecurity扩展（它针对SpringSecurity提供了更多额外的解析），因此使用thymeleaf的文件要添加
```html
<html lang="en" xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
```

**创建config包，里面包含三个配置类**

`MvcConfiguration`配置类

前期工作就先把thymeleaf视图解析器和静态资源配置好就行
再一个就是扫描controller包

```java
@ComponentScan("book.manager.controller")
@Configuration
@EnableWebMvc
public class MvcConfiguration implements WebMvcConfigurer {

    //我们需要使用ThymeleafViewResolver作为视图解析器，并解析我们的HTML页面
    @Bean
    public ThymeleafViewResolver thymeleafViewResolver(@Autowired SpringTemplateEngine springTemplateEngine){
        ThymeleafViewResolver resolver = new ThymeleafViewResolver();
        resolver.setOrder(1);
        resolver.setCharacterEncoding("UTF-8");
        resolver.setTemplateEngine(springTemplateEngine);
        return resolver;
    }

    //配置模板解析器
    @Bean
    public SpringResourceTemplateResolver templateResolver(){
        SpringResourceTemplateResolver resolver = new SpringResourceTemplateResolver();
         resolver.setCharacterEncoding("UTF-8");
        resolver.setSuffix(".html");
        resolver.setPrefix("/WEB-INF/template/");
        return resolver;
    }

    //配置模板引擎Bean
    @Bean
    public SpringTemplateEngine springTemplateEngine(@Autowired ITemplateResolver resolver){
        SpringTemplateEngine engine = new SpringTemplateEngine();
        engine.setTemplateResolver(resolver);
        engine.addDialect(new SpringSecurityDialect());   //添加针对于SpringSecurity的方言
        return engine;
    }

    //开启静态资源处理
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    //静态资源路径配置
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**").addResourceLocations("/WEB-INF/static/");
    }
}
```

`RootConfiguration`配置类

前期工作先不用管这个
扫描service包

```java
@ComponentScans({
        @ComponentScan("book.manager.service")
})
@Configuration
public class RootConfiguration {
}
```

`SecurityConfiguration`配置类

创建好就行，先不用管

```java
@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    //继承WebSecurityConfigurerAdapter，之后会进行配置
}
```

**创建initializer包**

`MvcInitializer`

```java
public class MvcInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{RootConfiguration.class, SecurityConfiguration.class};
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{MvcConfiguration.class};
    }

    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}
```

`SecurityInitializer`

```java
public class SecurityInitializer extends AbstractSecurityWebApplicationInitializer {
    //不用重写任何内容
    //这里实际上会自动注册一个Filter，SpringSecurity底层就是依靠N个过滤器实现的，之后再探讨
}
```

**创建controller包**

前期工作就先弄一个`PageController`用于页面响应

```java
@Controller
public class PageController {
    @RequestMapping("/index")
    public String index(){
        return "index";
    }
}
```

**创建mapper包，entity包**

**创建service包，其下再创建impl包**

**WEB-INF包下创建static目录和template目录**

static目录：将前端模板里的static目录下的内容复制到这里

template目录：就先将前端模板里的index.html文件复制进来

## 登录，退出，注册功能

**目标效果**
使用数据库认证进行登录：用户信息都存放在users表里，密码是加密存储

登录功能：
>1. 登录界面主要有“登录”，“记住我”和“跳转至注册界面”这三个功能
>2. 登录成功后还能获取到此账户的角色（“admin”or“user”），为后续不同角色展示不同页面做铺垫

注册功能：
>1. 在登录界面点击“注册”就会进入注册页面
>2. 正确填入“姓名”，“性别”，“年级”和“密码”就可以成功注册账户
>3. 管理员“admin”由我自己直接在数据库进行分配，通过注册创建的账户只能是用户“user”角色

**创建MainTest测试类**

```java
public class MainTest {

    @Test
    public void test(){
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        System.out.println(encoder.encode("123456"));
    }
}
```

**`PageController`中添加方法**

```java
@Controller
public class PageController {
    
    @RequestMapping("/index")
    public String index(){
        return "index";
    }
    
    @RequestMapping("/login")
    public String login(){
        return "login";
    }

    @RequestMapping("/register")
    public String register(){
        return "register";
    }
}
```

**将前端模板的login.html,register.html文件复制到template目录**

**修改login.html**

1. 找到登录from表单，添加`action="api/auth/login"`
```html
<form action="api/auth/login" method="post">
```
2. 用户名和密码输入框分别添加`name="username"`和`name="password"`
```html
<input name="username" type="text" placeholder="用户名" class="ad-input">
```
```html
<input name="password" type="password" placeholder="密码" class="ad-input">
```
3. 修改一下“记住我”勾选框`name`，将名称修改为与`rememberMeParameter("remember")`一致。如果上面没有配置名称，那么默认使用"remember-me"作为名称：
```html
<input type="checkbox" name="remember" class="ad-checkbox">
```
4. 将“点击注册”的链接设置为"register"
```html
 <p class="ad-register-text">还没有注册? <a href="register">点击注册</a></p>
```
5. 按需求修改文件即可

**index.html页面中找一个链接标签，将其设置为"api/auth/logout"**
```html
<li>
    <a href="api/auth/logout">
    <i class="fas fa-sign-out-alt"></i> 退出登录
    </a>
</li>
```

**修改register.html文件**
1. 对应的输入框添加对应的`name`属性
```html
<input name="username" type="text" placeholder="姓名" class="ad-input">
<input name="sex" type="text" placeholder="性别" class="ad-input">
<input name="grade" type="text" placeholder="年级" class="ad-input">
<input name="password" type="password" placeholder="密码" class="ad-input">
```
2. from表单添加`action="api/auth/register"`
```html
 <form action="api/auth/register" method="post">
```
3. 登录按钮链接设置为"login"
```html
<p class="ad-register-text">已经有账号了？<a href="login">登录</a></p>
```

**entity包下创建`AuthUser`类**

```java
@Data
@AllArgsConstructor
public class AuthUser {
    int id;
    String name;
    String password;
    String role;
}
```

**mapper包下创建`UserMapper`类**

注册用户：填入的是学生的基本信息，对应student表要添加，而给它分配一个账户，所以users表也要添加

```java
@Mapper
public interface UserMapper {

    @Select("select * from users where name = #{username}")
    AuthUser getPasswordByUsername(String username);

    @Options(useGeneratedKeys = true,keyColumn = "id",keyProperty = "id")
    @Insert("insert into users(name,password,role) values(#{name},#{password},#{role})")s
    int registerUser(AuthUser user);

    @Insert("insert into student(uid,name,grade,sex) values(#{uid},#{name},#{grade},#{sex})")
    int addStudentInfo(@Param("uid")int uid,@Param("name")String name,@Param("grade")String grade,@Param("sex")String sex);
}
```

**service包下的impl包下创建`UserAuthService`类**

这个类实现SpringSecurity提供的`UserDetailsService`接口

`loadUserByUsername(String s)`:获取输入的密码s，从数据库找对应的用户，如果找不到，登录失败；找到，则登录成功，并返回一个`UserDetails`对象

```java
@Service
public class UserAuthService implements UserDetailsService {

    @Resource
    UserMapper mapper;

    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        AuthUser user = mapper.getPasswordByUsername(s);
        if(user == null)
            throw new UsernameNotFoundException("登录失败，用户名或密码错误！");
        return User
                .withUsername(user.getName())
                .password(user.getPassword())
                .roles(user.getRole())
                .build();
    }
}
```

**service包下创建`AuthService`接口**

```java
public interface AuthService {
    public boolean register(String name,String sex,String grade,String password);
}
```

**service包下的impl包下创建`AuthServiceImpl`实现`AuthService`接口**

`按register(String name, String sex, String grade, String password)`的信息创建一个`AuthUser`对象，将这个对象添加到users表和student表中

*注意：`register()`添加了`@Transactional`进行事务管理，在后面`RootConfiguration`会说明为什么这么做*

```java
@Service
public class AuthServiceImpl implements AuthService {
    @Resource
    UserMapper mapper;

    @Transactional
    @Override
    public boolean register(String name, String sex, String grade, String password) {
        BCryptPasswordEncoder encoder=new BCryptPasswordEncoder();
        AuthUser user=new AuthUser(0,name,encoder.encode(password),"user");
        if(mapper.registerUser(user)<=0)
            throw new RuntimeException("用户基本信息添加失败");
        if(mapper.addStudentInfo(user.getId(),name,grade,sex)<=0)
            throw new RuntimeException("学生详细详细插入失败");
        return true;
    }
}
```

**controller包下创建`AuthController`**

```java
@Controller
@RequestMapping("/api/auth")
public class AuthController {
    @Resource
    AuthService service;
    
    @RequestMapping(value = "/register",method = RequestMethod.POST)
    public String register(@RequestParam("username")String name,
                           @RequestParam("sex")String sex,
                           @RequestParam("grade")String grade,
                           @RequestParam("password")String password){
        service.register(name,sex,grade,password);
        return "redirect:/login";
    }
}
```

**`SecurityInitializer`中添加`beforeSpringSecurityFilterChain`**

这个是为了添加到数据库的信息不出现乱码

```java
@Override
    protected void beforeSpringSecurityFilterChain(ServletContext servletContext) {
        servletContext.addFilter("characterEncodingFilter",new CharacterEncodingFilter("UTF-8",true))
                .addMappingForUrlPatterns(null,false,"/*");
    }
```

**`RootConfiguration`配置类**

1. 配置数据源（相当于Mybatis的xml文件）
2. `transactionManager(@Autowired DataSource dataSource)`开启Spring事务管理（发现异常就会回滚，无法添加到数据库）,为了注册时不乱填（性别是枚举类型）

```java
@ComponentScans({
        @ComponentScan("book.manager.service")
})
@MapperScan("book.manager.mapper")
@Configuration
@EnableTransactionManagement
public class RootConfiguration {
    @Bean
    public DataSource dataSource(){
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/book_manage");
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

    @Bean
    public TransactionManager transactionManager(@Autowired DataSource dataSource){
        return new DataSourceTransactionManager(dataSource);
    }
}
```

**`SecurityConfiguration`配置类**

1. `jdbcRepository(@Autowired DataSource dataSource)`是为了实现“记住我”功能
2. `configure(HttpSecurity http)`是最核心的：分权+登录/退出请求配置+关闭csrf+配置“记住我”
3. `configure(AuthenticationManagerBuilder auth)`使用自定义的Service实现类（`UserAuthService`）进行数据库验证

```java
@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Resource
    UserAuthService service;

    @Resource
    PersistentTokenRepository repository;

    @Bean
    public PersistentTokenRepository jdbcRepository(@Autowired DataSource dataSource){
        JdbcTokenRepositoryImpl repository = new JdbcTokenRepositoryImpl();  //使用基于JDBC的实现
        repository.setDataSource(dataSource);   //配置数据源
        //repository.setCreateTableOnStartup(true);   //启动时自动创建用于存储Token的表（建议第一次启动之后删除该行）
        return repository;
    }

    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/static/**","/login","/register","/api/auth/**").permitAll()
                .anyRequest().hasAnyRole("user","admin")   //除了上面以外的所有内容，只能是admin/user访问
                .and()
                .formLogin()       //配置Form表单登陆
                .loginPage("/login")       //登陆页面地址（GET）
                .loginProcessingUrl("/api/auth/login")    //form表单提交地址（POST）
                .defaultSuccessUrl("/index",true)    //登陆成功后跳转的页面，也可以通过Handler实现高度自定义
                .permitAll()    //登陆页面也需要允许所有人访问
                .and()
                .logout()
                .logoutUrl("/api/auth/logout")    //退出登陆的请求地址
                .logoutSuccessUrl("/login")    //退出后重定向的地址
                .and()
                .csrf().disable()
                .rememberMe()
                .rememberMeParameter("remember")
                .tokenRepository(repository)
                .tokenValiditySeconds(60 * 60 * 24 * 7) ; //Token的有效时间（秒）默认为14天
    }

    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
                .userDetailsService(service)   //使用自定义的Service实现类进行验证
                .passwordEncoder(new BCryptPasswordEncoder());   //依然使用BCryptPasswordEncoder
    }
}
```

**给`MvcConfiguration`配置类添加`@EnableGlobalMethodSecurity(prePostEnabled = true)`注解**

这个是为了能为`PageController`里的方法添加`@PreAuthorize`注解，进行权限控制

## 主页面框架搭建

**目标效果**
1. 登陆成功后的主页面侧边栏：admin显示“借阅管理”和“图书管理”；user则显示“图书借阅”和“借阅管理”，下一阶段再细化角色页面（不同角色，点击侧边栏会呈现不同页面）
2. 将成功登录的用户的姓名和角色显示在主页面上

**修改以下`PageController`的响应index方法**

1. 给`PageController`注入`UserMapper`对象
2. 完善`index()`方法功能，使其获取登录者的姓名和角色，显示在前端

```java
@Resource
UserMapper mapper;

@RequestMapping("/index")
public String index(HttpSession session, Model model) {
    AuthUser user=(AuthUser) session.getAttribute("user");
    if(user==null){
        Authentication authentication= SecurityContextHolder.getContext().getAuthentication();
        user=mapper.getPasswordByUsername(authentication.getName());
        session.setAttribute("user",user);
    }
    model.addAttribute("user",user);
    return "index";
}
```

**修改index.html文件**

1. 找个地方显示登录者的姓名和角色（配合上面已经完善的`index()`）
```html
<div class="drop-down-header">
    <h4 th:text="${user.getName()}">John Brown</h4>
    <p th:text="${user.getRole().equals('user')?'普通学生':'管理员'}">UI | UX Designer</p>
</div>
```

2. 用`sec:authorize="hasRole()"`为侧边栏设定权限
   admin则显示“借阅管理”和“图书管理”；user则显示“图书借阅”和“借阅管理”

```html
<li sec:authorize="hasRole('admin')">
    ...
</li>
<li sec:authorize="hasRole('user')">
    ...
</li>
```

## 角色页面划分

**目标效果**
用户分为admin和user这两个角色，不同角色看到的页面是不一样的

admin角色的页面：侧边栏有“借阅管理”和“图书管理”这两个选项，点击后即可显示对应的页面

user角色的页面：侧边栏有“借阅管理”和“图书借阅”这两个选项，点击后即可显示对应的页面

综上所述，一共要做出四个不同的页面，而这四个页面的共同部分可以提取出来放到一个模板文件里

**注意：这一阶段，对项目结构有多处调整，主要是细分职责以及html文件提取模板**

**template目录下创建admin目录和user目录**
现在，管理员的页面放在admin下，普通用户的页面放在user下

**template目录下创建template.html作为模板文件，存放各个页面共有的部分**

1. 将index.html文件里，所有页面的共有的部分（侧边栏，上方栏）复制到template.html文件里，并添加`th:fragment`属性
```html
<header class="header-wrapper main-header" th:fragment="title"...>
<aside class="sidebar-wrapper" th:fragment="aside"...>
```
2. 将上一步index.html选中的部分删除，改为引入template.html模板
```html
<div th:replace="template.html::title"></div>
<div th:replace="template.html::aside"></div>
```
3. 将这个改后的index.html放到admin和user目录下，分别做admin用户和user用户的“借阅管理”页面。也就是原来页面主要内容Container部分删去，根据需求自己找资源加进去
4. 在admin目录和user目录下都创建一个book.html文件，仿照index.html那样分别做admin的“图书管理”和user的“借阅图书”页面

**细化controller包：包下创建api包和page包**

将之前的`AuthController`放到api包下，改名为`AuthApiController`
将`PageController`改名为`AuthPageController`，并放到page包下
page包下创建`AdminPageController`和`UserPageController`类

**修改``AuthPageController``类**

* 给类上添加`@RequestMapping("/page/auth")`
* 将`index()`方法删掉（因为现在要分为admin页面和user页面了），只保留登录和注册的相应页面方法（也就是主页面交给`AdminPageController`和`UserPageController`去相应）

```java
@Controller
@RequestMapping("/page/auth")
public class AuthPageController {
    @RequestMapping("/login")
    public String login(){
        return "login";
    }

    @RequestMapping("/register")
    public String register(){
        return "register";
    }
}
```

**结构调整了一下之后，“前端显示姓名和角色”这个功能要重新写了**

`AuthService`添加方法
```java
AuthUser findUser(HttpSession session);
```

`AuthServiceImpl`中实现这个新添加的方法
```java
 public AuthUser findUser(HttpSession session){
        AuthUser user=(AuthUser) session.getAttribute("user");
        if(user==null){
            Authentication authentication= SecurityContextHolder.getContext().getAuthentication();
            user=mapper.getPasswordByUsername(authentication.getName());
            session.setAttribute("user",user);
        }
       return user;
    }
```

**`AdminPageController`类**

* `index()`响应admin角色的“借阅管理”页面
* `book()`响应admin角色的“图书管理”页面
```java
@Controller
@RequestMapping("/page/admin")
public class AdminPageController {

    @Resource
    AuthService service;

    @RequestMapping("/index")
    public String index(HttpSession session, Model model){
        model.addAttribute("user",service.findUser(session));
        return "/admin/index";
    }

    @RequestMapping("/book")
    public String book(HttpSession session, Model model){
        model.addAttribute("user",service.findUser(session));
        return "/admin/book";
    }
}
```

**`UserPageController`类**

就是复制`AdminPageController`类的代码，只是前缀路径和方法返回路径改一下就行
* `index()`响应user角色的“借阅管理”页面
* `book()`响应user角色的“图书借阅”页面
```java
@Controller
@RequestMapping("/page/user")
public class UserPageController {
    @Resource
    AuthService service;

    @RequestMapping("/index")
    public String index(HttpSession session, Model model){
        model.addAttribute("user",service.findUser(session));
        return "/user/index";
    }

    @RequestMapping("/book")
    public String book(HttpSession session, Model model){
        model.addAttribute("user",service.findUser(session));
        return "/user/book";
    }
}
```

**修改下`SecurityConfiguration`类**

“前端显示姓名和角色”这一功能的代码调整和各个请求地址都进行了调整，因此要修改一下

* 注入`UserMapper`和`UserAuthService`bean对象
* `configure()`重新配置一下
* 添加`onAuthenticationSuccess()`方法用于判断登录者的角色，以此来决定登录成功后重定向到哪个页面
```java
@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    @PostConstruct
    public void init(){
        SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_INHERITABLETHREADLOCAL);
    }

    @Resource
    UserMapper mapper;
    @Resource
    UserAuthService service;

    @Resource
    PersistentTokenRepository repository;

    @Bean
    public PersistentTokenRepository jdbcRepository(@Autowired DataSource dataSource){
        JdbcTokenRepositoryImpl repository = new JdbcTokenRepositoryImpl();  //使用基于JDBC的实现
        repository.setDataSource(dataSource);   //配置数据源
        //repository.setCreateTableOnStartup(true);   //启动时自动创建用于存储Token的表（建议第一次启动之后删除该行）
        return repository;
    }

    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/static/**","/page/auth/**","/api/auth/**").permitAll()
                .antMatchers("page/user/**").hasRole("user")
                .antMatchers("/page/admin/**").hasRole("admin")
                .anyRequest().hasAnyRole("user","admin")  
                .and()
                .formLogin()       //配置Form表单登陆
                .loginPage("/page/auth/login")       //登陆页面地址（GET）
                .loginProcessingUrl("/api/auth/login")    //form表单提交地址（POST）
                .successHandler(this::onAuthenticationSuccess)
                .and()
                .logout()
                .logoutUrl("/api/auth/logout")    //退出登陆的请求地址
                .logoutSuccessUrl("/login")    //退出后重定向的地址
                .and()
                .csrf().disable()
                .rememberMe()
                .rememberMeParameter("remember")
                .tokenRepository(repository)
                .tokenValiditySeconds(60 * 60 * 24 * 7) ; //Token的有效时间（秒）默认为14天
    }

    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
                .userDetailsService(service)   //使用自定义的Service实现类进行验证
                .passwordEncoder(new BCryptPasswordEncoder());   //依然使用BCryptPasswordEncoder
    }

    private void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
        HttpSession session=httpServletRequest.getSession();
        AuthUser user=mapper.getPasswordByUsername(authentication.getName());
        session.setAttribute("user",user);
        if(user.getRole().equals("admin")){
            httpServletResponse.sendRedirect("/bookmanager/page/admin/index");
        }else {
            httpServletResponse.sendRedirect("/bookmanager/page/user/index");//默认显示的是user页面
        }
    }
}
```

**上述操作后，发现登录界面和注册界面的特效渲染全没了，这里要修改下login.html和register.html文件**
1. 重新配置静态资源
```html
<link rel="stylesheet" type="text/css" href="static/css/fonts.css">
<link rel="stylesheet" type="text/css" href="static/css/bootstrap.min.css">
<link rel="stylesheet" type="text/css" href="static/css/auth.css">
```
改成
```html
<link rel="stylesheet" type="text/css" th:href="@{static/css/fonts.css}">
<link rel="stylesheet" type="text/css" th:href="@{/static/css/bootstrap.min.css}">
<link rel="stylesheet" type="text/css" th:href="@{/static/css/auth.css}">
```
1. 登录/注册界面的一个图片给搞回来
```html
<div class="ad-auth-img">
    <img src="static/picture/auth-img1.png" alt="">
</div>
```
改成
```html
<div class="ad-auth-img">
    <img th:src="@{/static/picture/auth-img1.png}" alt="">
</div>
```
1. login.html的from表单重新配置一下
```html
<form th:action="api/auth/login" method="post">
```
改成
```html
<form th:action="@{/api/auth/login}" method="post">
```
1. register.html的from表单重新配置一下
```html
<form action="api/auth/register" method="post">
```
改成
```html
<form th:action="@{/api/auth/register}" method="post">
```

**同理，登录进去之后的主页面的特效渲染也没了。只需和上面那样修改（改静态资源和图片src）admin目录下的index.html,book.html,user目录下的index.html,book.html和模板文件template.html**
注意：template.html中侧边栏的链接也要改
```html
<a th:href="@{/page/admin/index}" class="active">
<a th:href="@{/page/admin/book}">
<a th:href="@{/page/user/index}" class="active">
<a th:href="@{/page/user/book}">
```

注意：template.html中退出登录的链接也要改
```html
 <a th:href="@{/api/auth/logout}">
```

## admin角色：图书列表，删除和添加

**目标效果**
admin角色点击“图书管理”，响应出带有图书列表的页面（book.html），上面显示图书的`ID`,`书名`,`简介`和`价格`,还可以对图书进行删除。另外还有一个“添加新图书”的按钮，点击后响应出对应的页面(add-book.html)，填写`书名`,`简介`和`价格`即可添加。

**entity包下创建`Book`实体类**

```java
@Data
public class Book {
    int bid;
    String title;
    String desc;
    double price;
}
```

**mapper包下创建`BookMapper`类**

```java
@Mapper
public interface BookMapper {
    @Select("select * from book")
    List<Book> allBook();//查找book表里的所有书籍

    @Delete("delete from book where bid = #{bid}")
    void deleteBook(int bid);

    @Insert("insert into book(title,`desc`,price) values(#{title},#{desc},#{price})")
    void addBook(@Param("title")String title, @Param("desc")String desc,@Param("price")double price);
}
```

**service包下创建`BookService`接口**

```java
public interface BookService {
    List<Book> getAllBook();
    void deleteBook(int bid);
    void addBook(String title,String desc,double price);
}

```

**service包下的impl包下创建`BookServiceImpl`类实现`BookService`接口**

```java
@Service
public class BookServiceImpl implements BookService {

    @Resource
    BookMapper mapper;
    @Override
    public List<Book> getAllBook() {
        return mapper.allBook();
    }

    @Override
    public void deleteBook(int bid) {
        mapper.deleteBook(bid);
    }

    @Override
    public void addBook(String title, String desc, double price) {
        mapper.addBook(title,desc,price);
    }
}
```

**api包下创建`AdminApiController`类**

```java
@Controller
@RequestMapping("/api/admin")
public class AdminApiController {

    @Resource
    BookService service;

    @RequestMapping(value = "/del-book",method = RequestMethod.GET)
    public String deleteBook(@RequestParam("id")int id){
        service.deleteBook(id);
        return "redirect:/page/admin/book";
    }

    @RequestMapping(value = "/add-book",method = RequestMethod.POST)
    public String addBook(@RequestParam("title")String title,
                          @RequestParam("desc")String desc,
                          @RequestParam("price")double price){
       service.addBook(title,desc,price);
        return "redirect:/page/admin/book";
    }
}
```

**修改admin目录下的book.html文件**

* 显示图书列表
```html
<table class="table table-styled mb-0">
    <thead>
        <tr>
            <th>#ID</th>
            <th>标题</th>
            <th>简介</th>
            <th>价格</th>
            <th>删除</th>
        </tr>
     </thead>
        <tbody>
            <tr th:each="book : ${bookList}">
            <td th:text="${book.getBid()}">1</td>
            <td th:text="${book.getTitle()}">书籍名称</td>
            <td th:text="${book.getDesc()}">书籍简介</td>
            <td th:text="${book.getPrice()}">价格</td>
            <td class="relative">
                <a class="action-btn " th:href="@{/api/admin/del-book(id=${book.getBid()})}"></a>
        </tbody>
</table>
```

* “添加新图书”按钮
```html
 <div class="add-group">
    <a class="ad-btn" th:href="@{/page/admin/add-book}">添加新图书</a>
</div>
```

**在admin目录下创建add-book.html文件，用于显示添加图书的页面**

提交表单
```html
<form class="separate-form" method="post" th:action="@{/api/admin/add-book}">
```

输入框的`name`属性要和代码里面一致
```html
<input class="form-control" type="text" placeholder="输入书籍名称" name="title">
<input class="form-control" type="text" placeholder="请输入一段书籍简介"  name="desc">
<input class="form-control" type="text" placeholder="输入书籍价格(￥)"  name="price">
```

## user角色：图书借阅和归还

**目标效果**
“图书借阅”（index.html）页面显示*没有被借走的书籍*列表，可以点击进行借阅；“借阅管理”页面(book.html)可以显示自己的借阅列表，可以点击进行还书

**entitty包下创建`Borrow`实体类**

```java
@Data
public class Borrow {
    int id;
    int sid;
    int bid;
    Data date;
}
```

### “图书借阅”页面

**`BookMapper`添加`borrowList()`**
```java
@Select("select * from borrow")
List<Borrow> borrowList();
```

**`BookService`接口添加`getAllBookWithoutBorrow()`**
```java
List<Book> getAllBookWithoutBorrow();
```

**`BookServiceImpl`实现`getAllBookWithoutBorrow()`**
```java
@Override
    public List<Book> getAllBookWithoutBorrow() {
        List<Book> books=mapper.allBook();
        List<Integer> borrows=mapper.borrowList()
                .stream()
                .map(Borrow::getBid)
                .collect(Collectors.toList());
        return books.
                stream().
                filter(book->!borrows.contains(book.getBid()))
                .collect(Collectors.toList());
    }
```

**`UserPageController`类中修改下对应的响应方法`index()`**
```java
@Resource
AuthService service;

@Resource
BookService bookService;

@RequestMapping("/index")
public String index(HttpSession session, Model model){
    model.addAttribute("user",service.findUser(session));
    model.addAttribute("bookList",bookService.getAllBookWithoutBorrow());
    return "/user/index";
}
```

### “图书借阅”页面点击按钮进行借阅

**`BookMapper`接口添加`addBorrow`**
```java
@Insert("insert into borrow(bid,sid,`time`) values (#{bid},#{sid},NOW())")
void addBorrow(@Param("bid")int bid,@Param("sid")int sid);
```

**`UserMapper`接口添加`getSidByUserId`**
```java
@Select("select sid from student where uid=#{uid}")
Integer getSidByUserId(int uid);
```

**`BookService`接口添加`borrowBook(int bid,int id)`**
```java
void borrowBook(int bid,int id);
```

**`BookServiceImpl`中实现`borrowBook(int bid,int id)`**
```java
@Resource
    BookMapper mapper;
@Override
    public void borrowBook(int bid,int id) {
        Integer sid=userMapper.getSidByUserId(id);
        if(sid==null)return;
        mapper.addBorrow(bid,sid);
    }
```

**api包下创建`UserApiController`**
```java
@Controller
@RequestMapping("/api/user")
public class UserApiController {
    @Resource
    BookService service;

    @RequestMapping(value = "/borrow-book",method = RequestMethod.GET)
    public String borrowBook(@RequestParam("id") int bid,
                             @SessionAttribute("user")AuthUser user){
        service.borrowBook(bid,user.getId());
        return "redirect:/page/user/book";
    }
}
```

### “借阅管理”页面

**`BookMapper`接口添加`borrowListBySid(int sid)`和`getBookById(int bid)`**
```java
@Select("select * from borrow where sid = #{sid}")
List<Borrow> borrowListBySid(int sid);
@Select("select * from book where bid=#{bid}")
Book getBookById(int bid);
```

**`BookService`接口添加`getAllBorrowedByID(int id)`**
```java
List<Book> getAllBorrowedByID(int id);
```

**`BookServiceImpl`实现`getAllBorrowedByID(int id)`**
```java
 @Override
    public List<Book> getAllBorrowedByID(int id) {
        Integer sid=userMapper.getSidByUserId(id);
        if(sid==null)return Collections.emptyList();
        return mapper.borrowListBySid(sid)
                .stream()
                .map(borrow -> mapper.getBookById(borrow.getBid()))
                .collect(Collectors.toList());
    }
```

**`UserPageController`修改下对应的响应方法`book()`**
```java
@RequestMapping("/book")
    public String book(HttpSession session, Model model){
        AuthUser user=service.findUser(session);
        model.addAttribute("user",service.findUser(session));
        model.addAttribute("bookList",bookService.getAllBorrowedByID(user.getId()));
        return "/user/book";
    }
```

### “借阅管理”页面点击按钮还书

**`BookMapper`接口添加`deleteBorrow()`**
```java
@Delete("delete from borrow where bid=#{bid} and sid=#{sid}")
void deleteBorrow(@Param("bid") int bid,@Param("sid") int sid);
```

**`BookService`接口添加`returnBook(int bid,int id)`**
```java
void returnBook(int bid,int id);
```

**`BookServiceImpl`实现`returnBook(int bid,int id)`**
```java
@Override
    public void returnBook(int bid, int id) {
        Integer sid=userMapper.getSidByUserId(id);
        if(sid==null)return;
        mapper.deleteBorrow(bid,sid);
    }
```

**`UserApiController`中添加`returnBook()`方法**
```java
 @RequestMapping(value = "/return-book",method = RequestMethod.GET)
    public String returnBook(@RequestParam("id") int bid,
                             @SessionAttribute("user")AuthUser user){
        service.returnBook(bid,user.getId());
        return "redirect:/page/user/book";
    }
```

### 综合以上代码对html文件进行修改

**“图书借阅”页面index.html**

主要就是列表显示和“借阅”按钮的链接
```html
<table class="table table-styled mb-0">
	<thead>
		<tr>
		    <th>#ID</th>
		    <th>标题</th>
		    <th>简介</th>
		    <th>价格</th>
		    <th>删除</th>									
		</tr>
	</thead>
	<tbody>
		<tr th:each="book : ${bookList}">
			<td th:text="${book.getBid()}">1</td>
			<td th:text="${book.getTitle()}">书籍名称</td>
			<td th:text="${book.getDesc()}">书籍简介</td>
			<td th:text="${book.getPrice()}">价格</td>
		    <td class="relative">
			<a class="action-btn " th:href="@{/api/user/borrow-book?id=}+${book.getBid()}">
            ...
```

**“借阅管理”页面book.html**

表格和index.html一样，就是“归还”按钮的链接改一下就行
```html
<a class="action-btn " th:href="@{/api/user/return-book?id=}+${book.getBid()}">
```

## admin角色：“借阅管理”页面

**目标效果**
上方展示图书管理系统的统计信息，下方主体显示出一张借阅信息表，反映“谁借阅了那本书”。

**entity包下创建`BorrowDetails`实体类**
```java
@Data
public class BorrowDetails {
    int id;
    String book_title;
    String user_name;
    Date time;
}
```

### 显示借阅信息列表

**`BookMapper`接口添加`borrowDetailsList()`**
```java
@Results({
        @Result(id=true,column="id",property = "id"),
        @Result(column="title",property = "book_title"),
        @Result(column="name",property = "user_name"),
        @Result(column="time",property = "time"),
    })
@Select("select * from borrow left join book on book.bid=borrow.bid "+
            "left join student on borrow.sid=student.sid")
List<BorrowDetails> borrowDetailsList();
```

**`BookService`接口添加`getBorrowDetailsList()`**
```java
List<BorrowDetails> getBorrowDetailsList();
```

**`BookServiceImpl`实现`getBorrowDetailsList()`**
```java
@Override
    public List<BorrowDetails> getBorrowDetailsList() {
        return mapper.borrowDetailsList();
    }
```

**`AdminPageController`修改对应的响应方法`index()`**
```java
  @RequestMapping("/index")
    public String index(HttpSession session, Model model){
        model.addAttribute("user",service.findUser(session));
        model.addAttribute("borrowList",bookService.getBorrowDetailsList());
        return "/admin/index";
    }
```

**index.html的列表**
```html
<table class="table table-styled mb-0">
	<thead>
		<tr>
			<th>借阅 ID</th>
		    <th>借阅人</th>
			<th>借阅书籍</th>
		    <th>借阅时间</th>
		</tr>
	</thead>
	<tbody>
        <tr th:each="borrow : ${borrowList}">
			<td th:text="${borrow.getId()}">1</td>
			<td th:text="${borrow.getUser_name()}">书籍名称</td>
			<td th:text="${borrow.getBook_title()}">书籍简介</td>
			<td th:text="${borrow.getTime()}">价格</td>
		</tr>
	</tbody>
</table>
```

### 显示统计信息

**entity包下创建`GlobalStat`实体类**
```java
@Data
@AllArgsConstructor
public class GlobalStat {
    int userCount;
    int bookCount;
    int borrowCount;
}
```

**`UserMapper`接口添加`getStudentCount()`**
```java
@Select("select count(*) from student")
int getStudentCount();
```

**`BookMapper`接口添加`getBookCount()`和`getBorrowCount()`**
```java
@Select("select count(*) from book")
int getBookCount();

@Select("select count(*) from borrow")
int getBorrowCount();
```

**service包下创建`StatService`接口**
```java
public interface StatService {
    GlobalStat getGlobalStat();
}
```

**impl包下创建`StatServiceImpl`实现`StatService`接口**
```java
@Service
public class StatServiceImpl implements StatService {
    @Resource
    UserMapper userMapper;
    @Resource
    BookMapper bookMapper;
    @Override
    public GlobalStat getGlobalStat() {
        return new GlobalStat(userMapper.getStudentCount(),
                bookMapper.getBookCount(),
                bookMapper.getBorrowCount());
    }
}
```

**`AdminPageController`修改对应的响应方法`index()`**
```java
@Resource
StatService statService;

@RequestMapping("/index")
public String index(HttpSession session, Model model){
    model.addAttribute("user",service.findUser(session));
    model.addAttribute("borrowList",bookService.getBorrowDetailsList());
    model.addAttribute("stat",statService.getGlobalStat());
    return "/admin/index";
}
```

**index.html文件：将统计信息展示到前端，以学生数量为例**
```html
<div class="icon-info-text">
	<h5 class="ad-title">当前已注册学生数量</h5>
	<h4 class="ad-card-title" th:text="${stat.getUserCount()}">66k</h4>
</div>
```

## 总结
对于一个功能可以分为前端和后端考量

后端
1. 需要用到哪些实体类
2. mapper层（Dao层）需要写哪些对应的方法
3. 在service层对应这类问题的接口里添加实现这一功能的所需的业务，然后再去对应的实现类里用合适的mapper层里的方法去实现这些业务
4. 最终的成果通过controller层反映出来

前端：根据后端的代码，通过修改html文件将数据展示到前端

**config包**
这个包的配置类的目的就是为了取代xml配置文件，需要有`@Configuration`
![img](https://img2018.cnblogs.com/blog/738818/201906/738818-20190617214214614-761905677.png)

`RootConfiguration`
用来配置根容器，负责service层和repository，因此需要
1.  `@ComponentScan`扫描service包
2.  `@MapperScan`扫描mapper包
3.  `@EnableTransactionManagement`打开Spring事务管理

`MvcConfiguration`
用来配置web容器，负责Controllers，ViewResolver和HanderMapping，因此需要：
1. `@ComponentScan`扫描controller包
2. `@EnableWebMvc`启用SpringMVC
   
`SecurityConfiguration`

**controller层**

* controller起到一个承前启后的作用：接收前台请求，调用需要的service并且接收service给出的数据回复给前台。
* 一个controller可以有多个service，controller调用service不暴露具体细节（调用的是接口而不是其实现类）

**service层**

* 一个service可以有多个mapper（实现这个业务需要一个或多个不同的数据库操作）
* 给到controller的仅仅是一个接口，真正的实现写在impl中

**mapper（DAO）层**

mapper就是做数据库有关的工作，将sql语言发送给数据库，然后将结果给到需要的service

**entity层**

定义的属性最好和数据库相应的列是同名的

**各层间的联动**
1. controller层的类要有`@Controller`,Mapper层的接口要有`@Mapper`,service层的实现类要有`@Service`
2. controller层想要用到哪个service的方法，就将对应service注册为bean；同样的，mapper（DAO）层想要用到哪个mapper的方法，就将对应的mapper注册为bean。
