# OAuth 2.0 

## 简述

Oauth2.0是目前流行的**授权机制**，用于授权第三方应用，获取数据。Oauth协议为用户资源的授权提供一个安全、开放并且简易的规范标准 。

Oauth不会使第三方触及到用户的账号信息(用户和密码)，也就是说第三方不需要使用用户的用户名和密码就可以获取到该用户的用户资源权限。【对于一个网站，想要兼容微信登录，如果直接在网站输入微信号及密码，就会将这些消息暴露给该网站，OAuth就是为了避免这种数据泄露的风险】

***
**角色划分**

* **资源拥有者**`resource owner`: 拥有被访问资源的用户，如昵称、头像这些资源的拥有者(用户只是将这些资源放到服务提供商的资源服务器中)【我们的微信号，QQ号】
* **客户端**`client`: 第三方应用，比如我们开发的网站
* **认证服务器**`Authorization server`: 用来进行用户认证并颁发token
* **资源服务器**`Resource server`：拥有被访问资源的服务器，需要通过token来确定是否有权限访问
* **服务提供商**`Service Provider`：认证服务和资源服务归属于一个机构，该机构就是服务提供商。

***

**握手流程**

![img/OAuth2.0握手流程.png](img/OAuth2.0握手流程.png)

（A）客户端要求用户给予授权。
（B）用户同意给予客户端授权。
（C）客户端使用上一步获得的授权，向认证服务器申请令牌。
（D）认证服务器对客户端进行认证以后，确认无误，同意发放令牌。
（E）客户端使用令牌，向资源服务器申请获取资源。
（F）资源服务器确认令牌无误，同意向客户端开放资源

## 四种授权模式

OAuth 2.0一共有四种授权模式：

1. **客户端模式Client Credentials**

   这是最简单的一种模式，直接向验证服务器请求一个Token（Token相当于是一个令牌，需要在验证服务器**User Account And Authentication**服务拿到令牌之后，才能去访问资源，比如用户信息、借阅信息等，这样资源服务器才能知道是谁以及是否成功登录了）

   当然，这里的前端页面只是一个例子，它还可以是其他任何类型的**客户端**，比如App、小程序甚至是第三方应用的服务。

   ![image-20230306235141113](https://s2.loli.net/2023/03/06/4i16wzqOnYeaB2c.png)

   虽然这种模式比较简便，但是已经失去了用户验证的意义，压根就不是给用户校验准备的，而是更适用于服务内部调用的场景。

2. **密码模式Resource Owner Password Credentials**

   密码模式相比客户端模式，就多了用户名和密码的信息，用户需要提供对应账号的用户名和密码，才能获取到Token。

   ![image-20230306235151635](https://s2.loli.net/2023/03/06/JEreS9nQD8ojMca.png)

   虽然看起来比较合理，但是会直接将账号和密码泄露给客户端，需要后台完全信任客户端。所以这种模式并不常见。

3. **隐式授权模式Implicit Grant**

   首先用户访问页面时，会重定向到认证服务器，接着认证服务器给用户一个认证页面，等待用户授权，用户填写信息完成授权后，认证服务器返回Token。

   ![image-20230306235200365](https://s2.loli.net/2023/03/06/MRxnKyWT3br5Zj2.png)

   它适用于没有服务端的第三方应用页面，并且相比前面一种形式，验证都是在验证服务器进行的，敏感信息不会轻易泄露，但是Token依然存在泄露的风险。

4. **授权码模式（Authrization Code）**

   这种模式是最安全的一种模式，很多App都是使用的这种模式。

   相比隐式授权模式，它并不会直接返回Token，而是返回授权码，真正的Token是通过应用服务器访问验证服务器获得的。在一开始的时候，应用服务器（客户端通过访问自己的应用服务器来进而访问其他服务）和验证服务器之间会共享一个`secret`，这个东西没有其他人知道，而验证服务器在用户验证完成之后，会返回一个授权码，应用服务器最后将授权码和`secret`一起交给验证服务器进行验证，并且Token也是在服务端之间传递，不会直接给到客户端。

   ![image-20230306235211335](https://s2.loli.net/2023/03/06/2EIPfirBOKbcndk.png)

   就算有人中途窃取了授权码，也毫无意义。因为，Token的获取必须同时携带授权码和secret，而`secret`第三方是无法得知的，并且Token不会直接丢给客户端，大大减少了泄露的风险。

## 应用场景

OAuth2.0 最初就是为了实现第三方授权，基于第三方应用访问用户信息的权限（本质上就是给别人调用自己服务接口的权限），比如使用微信/QQ/微博账号授权登录我们这个第三方应用/提供微信支付服务...
这一部分的使用主要去看服务供应商的API文档
微信授权：https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/Wechat_Login.html

除此之外，还可以通过OAuth2.0 来实现单点登录，只是少了资源服务器这一角色，客户端就是我们的整个系统。
>分布式架构出现后，一个系统被拆分为多个系统
单点登录就是，用户只需在一个系统登录，各个系统即可感知该用户已经登录。

# 实现SSO单点登录

## 共享Session

>以图书管理系统为例，采用分布式架构后，系统拆分为用户服务，借阅服务和图书服务

从前的单体应用，服务器是如何判定一个请求是来自哪个用户的呢？

* 首先浏览器向服务端发送请求，访问网站。
* 服务端收到请求后，会创建一个SESSION ID，并暂时存储在服务端，然后会发送给浏览器作为Cookie保存。
* 之后浏览器会一直携带此Cookie访问服务器，服务器根据请求携带的Cookie中的SESSION ID判断是哪个用户。

现在采用分布式的系统，在用户服务进行登录之后，Session中的用户数据只会在用户服务的应用中保存，而在其他服务中，并没有对应的信息。也就是说，其他服务并没有完成登录。

![img/Session实现单点登录1.png](img/Session实现单点登录1.png)

现在的目标就是使所有的服务都能够同步这些Session信息，从而实现在用户服务登录之后其他服务都能知道
有以下两种方案：

方案一： 在每台服务器上都复制一份Session。这种做法在时间和空间上的开销不经济。

方案二： 将Session移出服务器，用统一存储来存放。比如在Redis或是MySQL中存放用户的Session信息，所有的服务器在需要获取Session信息时，统一访问Redis或是MySQL即可

![image-20230306234940672](https://s2.loli.net/2023/03/06/pqZolFN6eIPza52.png)

***

**实现2号方案**，使用Redis作为Session统一存储

这里就只使用Nacos就行了

把Nacos的包导入一下，然后进行一些配置

然后为每个服务都添加验证机制，首先导入依赖：

```xml
<!--  SpringSession Redis支持  -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
<!--  添加Redis的Starter  -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

使用SpringSecurity框架作为权限校验框架：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

在每个服务都编写一下对应的配置文件：

```yaml
spring:
  session:
  	# 存储类型修改为redis
    store-type: redis
  redis:
  	# Redis服务器的信息，该咋写咋写
    host: 1.14.121.107
```

这样，默认情况下，每个服务的接口都会被SpringSecurity所保护，只有登录成功之后，才可以被访问。

打开Nacos，看到三个服务都正常注册了
现在直接去访问图书服务：访问失败，重定向到登陆页面。同样的方式去访问其他服务，也是一样的效果。

由于现在是统一Session存储，那么就可以在任意一个服务登录之后，其他服务都可以正常访问

可以查看一下Redis服务器中是不是存储了Session信息

虽然看起来好像确实没啥问题了，但是借阅服务炸了：
在RestTemplate进行远程调用的时候，由于请求没有携带对应SESSION的Cookie，所以导致验证失败，访问不成功，返回401。
所以虽然这种方案看起来比较合理，但是在实际使用中，还是存在一些不便。

>虽然统一存储解决了Session共享问题，但是就算实现了Session共享，依然存在一些问题。
由于每个服务都有自己的验证模块，实际上整个系统是存在冗余功能的、同时还有上面出现的问题

## Oauth2.0方案

![img/单点登录.png](img/单点登录.png)

之前的登录模式称为多点登录，而通过OAuth2.0，可以真正实现单点登录

## 搭建验证服务器

第一步就是最重要的，我们需要搭建一个验证服务器，它是我们进行权限校验的核心，验证服务器有很多的第三方实现也有Spring官方提供的实现，这里我们使用Spring官方提供的验证服务器。

这里我们将最开始保存好的项目解压，就重新创建一个新的项目，首先我们在父项目中添加最新的SpringCloud依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>2021.0.1</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

接着创建一个新的模块`auth-service`，添加依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    
    <!--  OAuth2.0依赖，不再内置了，所以得我们自己指定一下版本  -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-oauth2</artifactId>
        <version>2.2.5.RELEASE</version>
    </dependency>
</dependencies>
```

接着我们修改一下配置文件：

```yaml
server:
  port: 8500
  servlet:
  	#为了防止一会在服务之间跳转导致Cookie打架（因为所有服务地址都是localhost，都会存JSESSIONID）
  	#这里修改一下context-path，这样保存的Cookie会使用指定的路径，就不会和其他服务打架了
  	#但是注意之后的请求都得在最前面加上这个路径
    context-path: /sso
```

接着我们需要编写一下配置类，这里需要两个配置类，一个是OAuth2的配置类，还有一个是SpringSecurity的配置类：

```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .anyRequest().authenticated()  //
                .and()
                .formLogin().permitAll();    //使用表单登录
    }
  
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        auth
                .inMemoryAuthentication()   //直接创建一个用户，懒得搞数据库了
                .passwordEncoder(encoder)
                .withUser("test").password(encoder.encode("123456")).roles("USER");
    }
  
  	@Bean   //这里需要将AuthenticationManager注册为Bean，在OAuth配置中使用
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
}
```

```java
@EnableAuthorizationServer   //开启验证服务器
@Configuration
public class OAuth2Configuration extends AuthorizationServerConfigurerAdapter {

    @Resource
    private AuthenticationManager manager;

    private final BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();

    /**
     * 这个方法是对客户端进行配置，一个验证服务器可以预设很多个客户端，
     * 之后这些指定的客户端就可以按照下面指定的方式进行验证
     * @param clients 客户端配置工具
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients
                .inMemory()   //这里我们直接硬编码创建，当然也可以像Security那样自定义或是使用JDBC从数据库读取
                .withClient("web")   //客户端名称，随便起就行
                .secret(encoder.encode("654321"))      //只与客户端分享的secret，随便写，但是注意要加密
                .autoApprove(false)    //自动审批，这里关闭，要的就是一会体验那种感觉
                .scopes("book", "user", "borrow")     //授权范围，这里我们使用全部all
                .authorizedGrantTypes("client_credentials", "password", "implicit", "authorization_code", "refresh_token");
                //授权模式，一共支持5种，除了之前我们介绍的四种之外，还有一个刷新Token的模式
                //这里我们直接把五种都写上，方便一会实验，当然各位也可以单独只写一种一个一个进行测试
                //现在我们指定的客户端就支持这五种类型的授权方式了
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) {
        security
                .passwordEncoder(encoder)    //编码器设定为BCryptPasswordEncoder
                .allowFormAuthenticationForClients()  //允许客户端使用表单验证，一会我们POST请求中会携带表单信息
                .checkTokenAccess("permitAll()");     //允许所有的Token查询请求
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints
                .authenticationManager(manager);
        //由于SpringSecurity新版本的一些底层改动，这里需要配置一下authenticationManager，才能正常使用password模式
    }
}
```

接着我们就可以启动服务器了：

![image-20230306235226961](https://s2.loli.net/2023/03/06/2FhnOKe1BorP5NE.png)

然后我们使用Postman进行接口测试，首先我们从最简单的客户端模式进行测试，客户端模式只需要提供id和secret即可直接拿到Token，注意需要再添加一个grant_type来表明我们的授权方式，默认请求路径为http://localhost:8500/sso/oauth/token：

![image-20230306235236376](https://s2.loli.net/2023/03/06/X81T7mz5gQK3iBk.png)

发起请求后，可以看到我们得到了Token，它是以JSON格式给到我们的：

![image-20230306235248456](https://s2.loli.net/2023/03/06/84IKgq2xdvBeLTm.png)

我们还可以访问 http://localhost:8500/sso/oauth/check_token 来验证我们的Token是否有效：

![image-20230306235257995](https://s2.loli.net/2023/03/06/SXD8FjzZn7ev2B3.png)

![image-20230306235309409](https://s2.loli.net/2023/03/06/B9TzojnUq4KvVPr.png)

可以看到active为true，表示我们刚刚申请到的Token是有效的。

接着我们来测试一下第二种password模式，我们还需要提供具体的用户名和密码，授权模式定义为password即可：

![image-20230306235318785](https://s2.loli.net/2023/03/06/jt5XPZKvRFqr73x.png)

接着我们需要在请求头中添加Basic验证信息，这里我们直接填写id和secret即可：

![image-20230306235327745](https://s2.loli.net/2023/03/06/K9ZpIv8SzcfsHd4.png)

可以看到在请求头中自动生成了Basic验证相关内容：

![image-20230306235335919](https://s2.loli.net/2023/03/06/JHxPKgFU5wY7SB8.png)

![image-20230306235345388](https://s2.loli.net/2023/03/06/F3WU7XhqridywVn.png)

响应成功，得到Token信息，并且这里还多出了一个refresh_token，这是用于刷新Token的，我们之后会进行讲解。

![image-20230306235355181](https://s2.loli.net/2023/03/06/zjuc2qxQmBas5r1.png)

查询Token信息之后还可以看到登录的具体用户以及角色权限等。

接着我们来看隐式授权模式，这种模式我们需要在验证服务器上进行登录操作，而不是直接请求Token，验证登录请求地址：http://localhost:8500/sso/oauth/authorize?client_id=web&response_type=token

注意response_type一定要是token类型，这样才会直接返回Token，浏览器发起请求后，可以看到熟悉而又陌生的界面，没错，实际上这里就是使用我们之前讲解的SpringSecurity进行登陆，当然也可以配置一下记住我之类的功能，这里就不演示了：

![image-20230306235441824](https://s2.loli.net/2023/03/06/OYeRQpEXFSoZMhc.png)

但是登录之后我们发现出现了一个错误：

![image-20230306235501474](https://s2.loli.net/2023/03/06/qLUkJFZau8eQ6WO.png)

这是因为登录成功之后，验证服务器需要将结果给回客户端，所以需要提供客户端的回调地址，这样浏览器就会被重定向到指定的回调地址并且请求中会携带Token信息，这里我们随便配置一个回调地址：

```java
@Override
public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    clients
            .inMemory()
            .withClient("web")
            .secret(encoder.encode("654321"))
            .autoApprove(false)
            .scopes("book", "user", "borrow")
            .redirectUris("http://localhost:8201/login")   //可以写多个，当有多个时需要在验证请求中指定使用哪个地址进行回调
            .authorizedGrantTypes("client_credentials", "password", "implicit", "authorization_code", "refresh_token");
}
```

接着重启验证服务器，再次访问：

![image-20230306235513330](https://s2.loli.net/2023/03/06/PnTwQhlYXDgBvry.png)

可以看到这里会让我们选择哪些范围进行授权，就像我们在微信小程序中登陆一样，会让我们授予用户信息权限、支付权限、信用查询权限等，我们可以自由决定要不要给客户端授予访问这些资源的权限，这里我们全部选择授予：

![image-20230306235521432](https://s2.loli.net/2023/03/06/p7nMEVZIKjXWAl5.png)

授予之后，可以看到浏览器被重定向到我们刚刚指定的回调地址中，并且携带了Token信息，现在我们来校验一下看看：

![image-20230306235530288](https://s2.loli.net/2023/03/06/g1JhS9WDfcz6QEK.png)

可以看到，Token也是有效的。

最后我们来看看第四种最安全的授权码模式，这种模式其实流程和上面是一样的，但是请求的是code类型：http://localhost:8500/sso/oauth/authorize?client_id=web&response_type=code

可以看到访问之后，依然会进入到回调地址，但是这时给的就是授权码了，而不是直接给Token，那么这个Token该怎么获取呢？

![image-20230306235536317](https://s2.loli.net/2023/03/06/da4WseDt172hbLV.png)

按照我们之前讲解的原理，我们需要携带授权码和secret一起请求，才能拿到Token，正常情况下是由回调的服务器进行处理，这里我们就在Postman中进行，我们复制刚刚得到的授权码，接口依然是`localhost:8500/sso/oauth/token`：

![image-20230306235545717](https://s2.loli.net/2023/03/06/e1Zdt9IP7vp2zMO.png)

可以看到结果也是正常返回了Token信息：

![image-20230306235554061](https://s2.loli.net/2023/03/06/qY5kxgBWSzMJXco.png)

这样我们四种最基本的Token请求方式就实现了。

最后还有一个是刷新令牌使用的，当我们的Token过期时，我们就可以使用这个refresh_token来申请一个新的Token：

![image-20230306235603731](https://s2.loli.net/2023/03/06/d2ojclCLB3mQu7D.png)

但是执行之后我们发现会直接出现一个内部错误：

![image-20230306235611413](https://s2.loli.net/2023/03/06/BcFMIg4NqCx8kdh.png)

![image-20230306235618838](https://s2.loli.net/2023/03/06/cA9WF1KxyUDZ8Bi.png)

查看日志发现，这里还需要我们单独配置一个UserDetailsService，我们直接把Security中的实例注册为Bean：

```java
@Bean
@Override
public UserDetailsService userDetailsServiceBean() throws Exception {
    return super.userDetailsServiceBean();
}
```

然后在Endpoint中设置：

```java
@Resource
UserDetailsService service;

@Override
public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
    endpoints
            .userDetailsService(service)
            .authenticationManager(manager);
}
```

最后再次尝试刷新Token：

![image-20230306235638503](https://s2.loli.net/2023/03/06/QWEwzpiq7FXnv3f.png)

OK，成功刷新Token，返回了一个新的。

## 基于@EnableOAuth2Sso实现

前面我们将验证服务器已经搭建完成了，现在我们就来实现一下单点登陆吧，SpringCloud为我们提供了客户端的直接实现，我们只需要添加一个注解和少量配置即可将我们的服务作为一个单点登陆应用，使用的是第四种授权码模式。

一句话来说就是，这种模式只是将验证方式由原本的默认登录形式改变为了统一在授权服务器登陆的形式。

首先还是依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
    <version>2.2.5.RELEASE</version>
</dependency>
```

我们只需要直接在启动类上添加即可：

```java
@EnableOAuth2Sso
@SpringBootApplication
public class BookApplication {
    public static void main(String[] args) {
        SpringApplication.run(BookApplication.class, args);
    }
}
```

我们不需要进行额外的配置类，因为这个注解已经帮我们做了：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@EnableOAuth2Client
@EnableConfigurationProperties({OAuth2SsoProperties.class})
@Import({OAuth2SsoDefaultConfiguration.class, OAuth2SsoCustomConfiguration.class, ResourceServerTokenServicesConfiguration.class})
public @interface EnableOAuth2Sso {
}
```

可以看到它直接注册了OAuth2SsoDefaultConfiguration，而这个类就是帮助我们对Security进行配置的：

```java
@Configuration
@Conditional({NeedsWebSecurityCondition.class})
public class OAuth2SsoDefaultConfiguration extends WebSecurityConfigurerAdapter {
  	//直接继承的WebSecurityConfigurerAdapter，帮我们把验证设置都写好了
    private final ApplicationContext applicationContext;

    public OAuth2SsoDefaultConfiguration(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }
```

接着我们需要在配置文件中配置我们的验证服务器相关信息：

```yaml
security:
  oauth2:
    client:
      #不多说了
      client-id: web
      client-secret: 654321
      #Token获取地址
      access-token-uri: http://localhost:8500/sso/oauth/token
      #验证页面地址
      user-authorization-uri: http://localhost:8500/sso/oauth/authorize
    resource:
      #Token信息获取和校验地址
      token-info-uri: http://localhost:8500/sso/oauth/check_token
```

现在我们就开启图书服务，调用图书接口：

![image-20230306235651433](https://s2.loli.net/2023/03/06/DrVSZtdKNCoMucx.png)

可以看到在发现没有登录验证时，会直接跳转到授权页面，进行授权登录，之后才可以继续访问图书服务：

![image-20230306235701725](https://s2.loli.net/2023/03/06/nsJGmxcOVYXDUqd.png)

那么用户信息呢？是否也一并保存过来了？我们这里直接获取一下SpringSecurity的Context查看用户信息，获取方式跟我们之前的视频中讲解的是一样的：

```java
@RequestMapping("/book/{bid}")
Book findBookById(@PathVariable("bid") int bid){
  	//通过SecurityContextHolder将用户信息取出
    SecurityContext context = SecurityContextHolder.getContext();
    System.out.println(context.getAuthentication());
    return service.getBookById(bid);
}
```

再次访问图书管理接口，可以看到：

![image-20230306235733777](https://s2.loli.net/2023/03/06/y1VYRC9tmOv854u.png)

这里使用的不是之前的UsernamePasswordAuthenticationToken也不是RememberMeAuthenticationToken，而是新的OAuth2Authentication，它保存了验证服务器的一些信息，以及经过我们之前的登陆流程之后，验证服务器发放给客户端的Token信息，并通过Token信息在验证服务器进行验证获取用户信息，最后保存到Session中，表示用户已验证，所以本质上还是要依赖浏览器存Cookie的。

接下来我们将所有的服务都使用这种方式进行验证，别忘了把重定向地址给所有服务都加上：

```java
@Override
public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    clients
            .inMemory()
            .withClient("web")
            .secret(encoder.encode("654321"))
            .autoApprove(true)   //这里把自动审批开了，就不用再去手动选同意了
            .scopes("book", "user", "borrow")
            .redirectUris("http://localhost:8101/login", "http://localhost:8201/login", "http://localhost:8301/login")
            .authorizedGrantTypes("client_credentials", "password", "implicit", "authorization_code", "refresh_token");
}
```

这样我们就可以实现只在验证服务器登陆，如果登陆过其他的服务都可以访问了。

但是我们发现一个问题，就是由于SESSION不同步，每次切换不同的服务进行访问都会重新导验证服务器去验证一次：

![image-20230306235744660](https://s2.loli.net/2023/03/06/7zbqlOrSCVRdQ4y.png)

这里有两个方案：

* 像之前一样做SESSION统一存储
* 设置context-path路径，每个服务单独设置，就不会打架了

但是这样依然没法解决服务间调用的问题，所以仅仅依靠单点登陆的模式不太行。

## 基于@EnableResourceServer实现

前面我们讲解了将我们的服务作为单点登陆应用直接实现单点登陆，那么现在我们如果是以第三方应用进行访问呢？这时我们就需要将我们的服务作为资源服务了，作为资源服务就不会再提供验证的过程，而是直接要求请求时携带Token，而验证过程我们这里就继续用Postman来完成，这才是我们常见的模式。

一句话来说，跟上面相比，我们只需要携带Token就能访问这些资源服务器了，客户端被独立了出来，用于携带Token去访问这些服务。

我们也只需要添加一个注解和少量配置即可：

```java
@EnableResourceServer
@SpringBootApplication
public class BookApplication {
    public static void main(String[] args) {
        SpringApplication.run(BookApplication.class, args);
    }
}
```

配置中只需要：

```yaml
security:
  oauth2:
    client:
    	#基操
      client-id: web
      client-secret: 654321
    resource:
    	#因为资源服务器得验证你的Token是否有访问此资源的权限以及用户信息，所以只需要一个验证地址
      token-info-uri: http://localhost:8500/sso/oauth/check_token
```

配置完成后，我们启动服务器，直接访问会发现：

![image-20230306235756676](https://s2.loli.net/2023/03/06/QiZmqznyMxNpETk.png)

这是由于我们的请求头中没有携带Token信息，现在有两种方式可以访问此资源：

* 在URL后面添加`access_token`请求参数，值为Token值
* 在请求头中添加`Authorization`，值为`Bearer +Token值`

我们先来试试看最简的一种：

![image-20230306235804196](https://s2.loli.net/2023/03/06/Np6PKCZD2kAdmtf.png)

另一种我们需要使用Postman来完成：

![image-20230306235814228](https://s2.loli.net/2023/03/06/ypR3G7DxsYicMQI.png)

添加验证信息后，会帮助我们转换成请求头信息：

![image-20230306235826416](https://s2.loli.net/2023/03/06/qPHDU1dXgC7srn3.png)

![image-20230306235833330](https://s2.loli.net/2023/03/06/6IeMvTcCKdfbUlV.png)

这样我们就将资源服务器搭建完成了。

我们接着来看如何对资源服务器进行深度自定义，我们可以为其编写一个配置类，比如我们现在希望用户授权了某个Scope才可以访问此服务：

```java
@Configuration
public class ResourceConfiguration extends ResourceServerConfigurerAdapter { //继承此类进行高度自定义

    @Override
    public void configure(HttpSecurity http) throws Exception {  //这里也有HttpSecurity对象，方便我们配置SpringSecurity
        http
                .authorizeRequests()
                .anyRequest().access("#oauth2.hasScope('lbwnb')");  //添加自定义规则
      					//Token必须要有我们自定义scope授权才可以访问此资源
    }
}
```

可以看到当没有对应的scope授权时，那么会直接返回`insufficient_scope`错误：

![image-20230306235844783](https://s2.loli.net/2023/03/06/5T4d39YkcZIomvD.png)

不知道各位是否有发现，实际上资源服务器完全没有必要将Security的信息保存在Session中了，因为现在只需要将Token告诉资源服务器，那么资源服务器就可以联系验证服务器，得到用户信息，就不需要使用之前的Session存储机制了，所以你会发现HttpSession中没有**SPRING_SECURITY_CONTEXT**，现在Security信息都是通过连接资源服务器获取。

接着我们将所有的服务都

但是还有一个问题没有解决，我们在使用RestTemplate进行服务间的远程调用时，会得到以下错误：

![image-20230306235853397](https://s2.loli.net/2023/03/06/k3LmR9E7UBtVA5x.png)

实际上这是因为在服务调用时没有携带Token信息，我们得想个办法把用户传来的Token信息在进行远程调用时也携带上，因此，我们可以直接使用OAuth2RestTemplate，它会在请求其他服务时携带当前请求的Token信息。它继承自RestTemplate，这里我们直接定义一个Bean：

```java
@Configuration
public class WebConfiguration {

    @Resource
    OAuth2ClientContext context;

    @Bean
    public OAuth2RestTemplate restTemplate(){
        return new OAuth2RestTemplate(new ClientCredentialsResourceDetails(), context);
    }
}
```

接着我们直接替换掉之前的RestTemplate即可：

```java
@Service
public class BorrowServiceImpl implements BorrowService {

    @Resource
    BorrowMapper mapper;

    @Resource
    OAuth2RestTemplate template;

    @Override
    public UserBorrowDetail getUserBorrowDetailByUid(int uid) {
        List<Borrow> borrow = mapper.getBorrowsByUid(uid);
        User user = template.getForObject("http://localhost:8101/user/"+uid, User.class);
        //获取每一本书的详细信息
        List<Book> bookList = borrow
                .stream()
                .map(b -> template.getForObject("http://localhost:8201/book/"+b.getBid(), Book.class))
                .collect(Collectors.toList());
        return new UserBorrowDetail(user, bookList);
    }
}
```

可以看到服务成功调用了：

![image-20230306235903496](https://s2.loli.net/2023/03/06/mvKqyJk7P1FCQSl.png)

现在我们来将Nacos加入，并通过Feign实现远程调用。

依赖还是贴一下，不然找不到：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2021.0.1.0</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

所有服务都已经注册成功了：

![image-20230306235926214](https://s2.loli.net/2023/03/06/BqkomFVGK7wv64X.png)

接着我们配置一下借阅服务的负载均衡：

```java
@Configuration
public class WebConfiguration {

    @Resource
    OAuth2ClientContext context;

    @LoadBalanced   //和RestTemplate一样直接添加注解就行了
    @Bean
    public OAuth2RestTemplate restTemplate(){
        return new OAuth2RestTemplate(new ClientCredentialsResourceDetails(), context);
    }
}
```

![image-20230306235934751](https://s2.loli.net/2023/03/06/PZkS8GyU1jhpIrz.png)

现在我们来把它替换为Feign，老样子，两个客户端：

```java
@FeignClient("user-service")
public interface UserClient {
    
    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") int uid);
}
```

```java
@FeignClient("book-service")
public interface BookClient {

    @RequestMapping("/book/{bid}")
    Book getBookById(@PathVariable("bid") int bid);
}
```

但是配置完成之后，又出现刚刚的问题了，OpenFeign也没有携带Token进行访问：

![image-20230306235944272](https://s2.loli.net/2023/03/06/EzWuaAJgNLi3sdF.png)

那么怎么配置Feign携带Token访问呢？遇到这种问题直接去官方查：https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/#oauth2-support，非常简单，两个配置就搞定：

```yaml
feign:
  oauth2:
  	#开启Oauth支持，这样就会在请求头中携带Token了
    enabled: true
    #同时开启负载均衡支持
    load-balanced: true
```

重启服务器，可以看到结果OK了：

![image-20230306235953555](https://s2.loli.net/2023/03/06/zgEJF7AeHw8iGIZ.png)

这样我们就成功将之前的三个服务作为资源服务器了，注意和我们上面的作为客户端是不同的，将服务直接作为客户端相当于只需要验证通过即可，并且还是要保存Session信息，相当于只是将登录流程换到统一的验证服务器上进行罢了。而将其作为资源服务器，那么就需要另外找客户端（可以是浏览器、小程序、App、第三方服务等）来访问，并且也是需要先进行验证然后再通过携带Token进行访问，这种模式是我们比较常见的模式。

## 使用jwt存储Token

官网：https://jwt.io

JSON Web Token令牌（JWT）是一个开放标准（[RFC 7519](https://tools.ietf.org/html/rfc7519)），它定义了一种紧凑和自成一体的方式，用于在各方之间作为JSON对象安全地传输信息。这些信息可以被验证和信任，因为它是数字签名的。JWT可以使用密钥（使用**HMAC**算法）或使用**RSA**或**ECDSA**进行公钥/私钥对进行签名。

实际上，我们之前都是携带Token向资源服务器发起请求后，资源服务器由于不知道我们Token的用户信息，所以需要向验证服务器询问此Token的认证信息，这样才能得到Token代表的用户信息，但是各位是否考虑过，如果每次用户请求都去查询用户信息，那么在大量请求下，验证服务器的压力可能会非常的大。而使用JWT之后，Token中会直接保存用户信息，这样资源服务器就不再需要询问验证服务器，自行就可以完成解析，我们的目标是不联系验证服务器就能直接完成验证。

JWT令牌的格式如下：

![image-20230307000004710](https://s2.loli.net/2023/03/07/Xu8lxYhKoJNr6it.png)

一个JWT令牌由3部分组成：标头(Header)、有效载荷(Payload)和签名(Signature)。在传输的时候，会将JWT的3部分分别进行Base64编码后用`.`进行连接形成最终需要传输的字符串。

* 标头：包含一些元数据信息，比如JWT签名所使用的加密算法，还有类型，这里统一都是JWT。
* 有效载荷：包括用户名称、令牌发布时间、过期时间、JWT ID等，当然我们也可以自定义添加字段，我们的用户信息一般都在这里存放。
* 签名：首先需要指定一个密钥，该密钥仅仅保存在服务器中，保证不能让其他用户知道。然后使用Header中指定的算法对Header和Payload进行base64加密之后的结果通过密钥计算哈希值，然后就得出一个签名哈希。这个会用于之后验证内容是否被篡改。

这里还是补充一下一些概念，因为很多东西都是我们之前没有接触过的：

* **Base64：**就是包括小写字母a-z、大写字母A-Z、数字0-9、符号"+"、"/"一共64个字符的字符集（末尾还有1个或多个`=`用来凑够字节数），任何的符号都可以转换成这个字符集中的字符，这个转换过程就叫做Base64编码，编码之后会生成只包含上述64个字符的字符串。相反，如果需要原本的内容，我们也可以进行Base64解码，回到原有的样子。

  ```java
  public void test(){
      String str = "你们可能不知道只用20万赢到578万是什么概念";
    	//Base64不只是可以对字符串进行编码，任何byte[]数据都可以，编码结果可以是byte[]，也可以是字符串
      String encodeStr = Base64.getEncoder().encodeToString(str.getBytes());
      System.out.println("Base64编码后的字符串："+encodeStr);
  
      System.out.println("解码后的字符串："+new String(Base64.getDecoder().decode(encodeStr)));
  }
  ```

  注意Base64不是加密算法，只是一种信息的编码方式而已。

* **加密算法：**加密算法分为对称加密和非对称加密，其中**对称加密（Symmetric Cryptography）**比较好理解，就像一把锁配了两把钥匙一样，这两把钥匙你和别人都有一把，然后你们直接传递数据，都会把数据用锁给锁上，就算传递的途中有人把数据窃取了，也没办法解密，因为钥匙只有你和对方有，没有钥匙无法进行解密，但是这样有个问题，既然解密的关键在于钥匙本身，那么如果有人不仅窃取了数据，而且对方那边的治安也不好，于是顺手就偷走了钥匙，那你们之间发的数据不就凉凉了吗。

  因此，**非对称加密（Asymmetric Cryptography）**算法出现了，它并不是直接生成一把钥匙，而是生成一个公钥和一个私钥，私钥只能由你保管，而公钥交给对方或是你要发送的任何人都行，现在你需要把数据传给对方，那么就需要使用私钥进行加密，但是，这个数据只能使用对应的公钥进行解密，相反，如果对方需要给你发送数据，那么就需要用公钥进行加密，而数据只能使用私钥进行解密，这样的话就算对方的公钥被窃取，那么别人发给你的数据也没办法解密出来，因为需要私钥才能解密，而只有你才有私钥。

  因此，非对称加密的安全性会更高一些，包括HTTPS的隐私信息正是使用非对称加密来保障传输数据的安全（当然HTTPS并不是单纯地使用非对称加密完成的，感兴趣的可以去了解一下）

  对称加密和非对称加密都有很多的算法，比如对称加密，就有：DES、IDEA、RC2，非对称加密有：RSA、DAS、ECC

* **不可逆加密算法：**常见的不可逆加密算法有MD5, HMAC, SHA-1, SHA-224, SHA-256, SHA-384, 和SHA-512, 其中SHA-224、SHA-256、SHA-384，和SHA-512我们可以统称为SHA2加密算法，SHA加密算法的安全性要比MD5更高，而SHA2加密算法比SHA1的要高，其中SHA后面的数字表示的是加密后的字符串长度，SHA1默认会产生一个160位的信息摘要。经过不可逆加密算法得到的加密结果，是无法解密回去的，也就是说加密出来是什么就是什么了。本质上，其就是一种哈希函数，用于对一段信息产生摘要，以**防止被篡改**。

  实际上这种算法就常常被用作信息摘要计算，同样的数据通过同样的算法计算得到的结果肯定也一样，而如果数据被修改，那么计算的结果肯定就不一样了。

这里我们就可以利用jwt，将我们的Token采用新的方式进行存储：

![image-20230307000016944](https://s2.loli.net/2023/03/07/W95CFKAmd1wfgSJ.png)

这里我们使用最简单的一种方式，对称密钥，我们需要对验证服务器进行一些修改：

```java
@Bean
public JwtAccessTokenConverter tokenConverter(){  //Token转换器，将其转换为JWT
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    converter.setSigningKey("lbwnb");   //这个是对称密钥，一会资源服务器那边也要指定为这个
    return converter;
}

@Bean
public TokenStore tokenStore(JwtAccessTokenConverter converter){  //Token存储方式现在改为JWT存储
    return new JwtTokenStore(converter);  //传入刚刚定义好的转换器
}
```

```java
@Resource
TokenStore store;

@Resource
JwtAccessTokenConverter converter;

private AuthorizationServerTokenServices serverTokenServices(){  //这里对AuthorizationServerTokenServices进行一下配置
    DefaultTokenServices services = new DefaultTokenServices();
    services.setSupportRefreshToken(true);   //允许Token刷新
    services.setTokenStore(store);   //添加刚刚的TokenStore
    services.setTokenEnhancer(converter);   //添加Token增强，其实就是JwtAccessTokenConverter，增强是添加一些自定义的数据到JWT中
    return services;
}

@Override
public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
    endpoints
            .tokenServices(serverTokenServices())   //设定为刚刚配置好的AuthorizationServerTokenServices
            .userDetailsService(service)
            .authenticationManager(manager);
}
```

然后我们就可以重启验证服务器了：

![image-20230307000026772](https://s2.loli.net/2023/03/07/C6OteoFghrxpYjQ.png)

可以看到成功获取了AccessToken，但是这里的格式跟我们之前的格式就大不相同了，因为现在它是JWT令牌，我们可以对其进行一下Base64解码：

![image-20230307000035581](https://s2.loli.net/2023/03/07/CUMkrRfgOthZKVz.png)

可以看到所有的验证信息包含在内，现在我们对资源服务器进行配置：

```yaml
security:
  oauth2:
    resource:
      jwt:
        key-value: lbwnb #注意这里要跟验证服务器的密钥一致，这样算出来的签名才会一致
```

然后启动资源服务器，请求一下接口试试看：

![image-20230307000043443](https://s2.loli.net/2023/03/07/kOpRlTB7SPtQa4y.png)

请求成功，得到数据：

![image-20230307000053404](https://s2.loli.net/2023/03/07/aicW89KezTSZ7f5.png)

注意如果Token有误，那么会得到：

![image-20230307000101304](https://s2.loli.net/2023/03/07/4wFZx8kNY5WHnvy.png)

***
