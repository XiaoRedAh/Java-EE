# JWT简介

官网：https://jwt.io/

JSON Web Token，是一个基于 RFC 7519 的开放数据标准，它定义了一种宽松且紧凑的数据组合方式。其作用是：**JWT是一种加密后数据载体，可在各应用之间进行数据传输**。JWT可以使用密钥（使用**HMAC**算法）或使用**RSA**或**ECDSA**进行公钥/私钥对进行签名。

![img/JWT示例.png](https://image.itbaima.net/images/253/image-20230902162497095.png)

JWT中一般涵盖了用户身份信息，每次访问时，server校验信息合法性即可。

# JWT组成

一个JWT通常有 HEADER (头)，PAYLOAD (有效载荷)和 SIGNATURE (签名)三个部分组成。在传输的时候，会将JWT的3部分分别进行Base64编码后，用`.`进行连接形成最终需要传输的字符串，格式如下：

```javascript
header.payload.signature
```

![img/JWT组成.png](https://image.itbaima.net/images/253/image-20230902161609043.png)



**一个简单的JWT案例**
> 注意三者之间有一个点号(“.”)相连

```html
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9   //header
.eyJ1c2VyX2luZm8iOlt7ImlkIjoiMSJ9LHsibmFtZSI6ImRhZmVpIn0seyJhZ2UiOiIxOCJ9XSwiaWF0IjoxNjgxNTcxMjU3LCJleHAiOjE2ODI3ODM5OTksImF1ZCI6InhpYW9mZWkiLCJpc3MiOiJkYWZlaSIsInN1YiI6ImFsbHVzZXIifQ  //payload
.v1TxJ0mngnVx4t9O3uibAHPSLUyMM7sUM06w8ODYjuE //signature
```

## Header

JWT的头部是json格式的，承载两部分信息：
**alg**：表示签名的算法，默认是 HMAC SHA256（写成 HS256）
**typ**： 表示令牌（token）的类型，JWT 令牌统一写为 `JWT`

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Base64编码后，构成了JWT的header：

```java
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

## Payload

Payload部分也是一个 JSON 对象，用来存放实际需要传递的有效信息。

**标准载荷**：有很多，建议使用，但不强制，对JWT信息作补充。

| 标准载荷              | 介绍                                   |
| --------------------- | -------------------------------------- |
| iss (issuer)          | 签发人（谁签发的）                     |
| exp (expiration time) | 过期时间，必须要大于签发时间           |
| sub (subject)         | 主题（用来做什么）                     |
| aud (audience)        | 受众(给谁用的)比如：http://www.xxx.com |
| nbf (Not Before)      | 生效时间                               |
| iat (Issued At)       | 签发时间                               |
| jti (JWT ID)          | 编号，JWT 的唯一身份标识               |

**自定义载荷**：可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息。但不建议添加敏感信息（账户密码之类的），因为该部分在客户端可解密。

```json
{
    "user_info": [
      {
        "id": "1"
      },
      {
        "name": "dafei"
      },
      {
        "age": "18"
      }
    ],
    "iat": 1681571257,
    "exp": 1682783999,
    "aud": "xiaofei",
    "iss": "dafei",
    "sub": "alluser"
}
```

Base64编码后，构成了JWT的payload：

```java
eyJ1c2VyX2luZm8iOlt7ImlkIjoiMSJ9LHsibmFtZSI6ImRhZmVpIn0seyJhZ2UiOiIxOCJ9XSwiaWF0IjoxNjgxNTcxMjU3LCJleHAiOjE2ODI3ODM5OTksImF1ZCI6InhpYW9mZWkiLCJpc3MiOiJkYWZlaSIsInN1YiI6ImFsbHVzZXIifQ
```

## Signature

Signature是对前两部分的签名，防止数据篡改。

首先，需要指定一个密钥（secret）。这个密钥**只有服务器知道，不能泄露给用户**。
然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名。

```java
signature = HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

Base64加密，构成了JWT的signature：

```java
l6JdYARw4IHmjliSbh9NP6ji1L15qVneWTJU5noQ-k8
```

**从公式中可以看到，计算签名的时候需要密钥。所以即使有人拿到我的JWT，没有密钥也解析不出来。或者有人改了我的JWT的前两部分，校验的时候也能发现**


# 在线生成/解析JWT

编码工具地址：https://tooltt.com/jwt-encode/

解码工具地址：https://tool.box3.cn/jwt.html

JWT官网也可以完成在线生成/解析JWT：https://jwt.io/

# JWT优缺点分析

**优点**

- `无状态`
  JWT 不需要在服务端存储任何状态，客户端可以携带 JWT 来访问服务端，从而使服务端变得无状态。这样，服务端就可以更轻松地实现扩展和负载均衡。
  >与之相对的，采用session是有状态的，需要在服务端建立session来缓存用户的信息，将session id给到客户端保管。客户端每次请求携带session id，都要去服务端对应的session中校验。

- `可自定义`：JWT的载荷可以自定义，按需添加信息。例如存储用户喜好、配置信息等等
  
- `可扩展性/跨平台性`
  JWT 有一套标准规范，因此很容易在不同平台和语言之间共享和解析。此外，开发人员可以根据需要自定义声明（claims）来实现更加灵活的功能。

- `可靠性`：JWT 使用数字签名来保证安全性，因此具有可靠性。

- `高效性`：JWT 不需要查询数据库

**缺点**

- `安全性取决于密钥管理`
  JWT 的安全性取决于密钥的管理，如果密钥被泄露或者被不当管理，那么 JWT 将会受到攻击。

- `无法撤销令牌`
  由于 JWT 是无状态的，一旦 JWT 被签发，就无法撤销。
  开发人员需要设计额外的机制来撤销 JWT，例如使用黑名单或者设置短期有效期等等。

- `需要缓存到客户端`
  由于 JWT 包含了用户信息和授权信息，因此 JWT 需要缓存到客户端，这意味着 JWT 有被攻击者窃取的风险。

- `载荷大小有限制`
  JWT 需要缓存到客户端，载荷大小一般不建议超过 1KB，会影响性能。

# JWT应用场景

## 一次性验证

用户注册成功后发一份激活邮件或者其他业务需要邮箱激活操作，都是可以使用jwt。

原因：
JWT时效性：让该链接具有时效性（比如约定2小时内激活）
JWT不可篡改性：防止篡改以激活其他账户

## RESTful API 的无状态认证

使用 JWT 来做 RESTful api 的身份凭证
当用户身份校验成功，客户端每次接口访问都带上JWT，服务端校验JWT合法性(是否篡改/是否过期等)

## 信息交换(传输)

JWT是在各方(项目间/服务间)之间安全传输信息的好方式。 
因为JWT可以签名：例如使用公钥/私钥对，所以可以确定发件人是他们自称的人。 
此外，由于使用标头和有效载荷计算签名，还可以验证内容是否未被篡改。

## JWT令牌登录

JWT 令牌登录也是一种应用场景，但也是JWT被诟病最多的地方，因为JWT令牌存在各种不安全。

* JWT令牌存储在客户端，容易泄露并被伪造身份搞破坏。
* JWT 被签发，就无法撤销，当破坏在进行时，后端无法马上禁止。

上面问题可通过监控异常JWT访问，设置黑名单 + 强制下线等方式尽量避免损失。

# SpringBoot项目使用JWT

## 导入依赖

具体选哪个JWT工具包，可以看官网推荐：https://jwt.io/libraries?language=Java
这里选择星星最多的jjwt

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.2.RELEASE</version>
    <relativePath/>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <scope>compile</scope>
    </dependency>

    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.11.5</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId> 
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```



## JWT常量类

```java
/**
 * 常量类
 */
public class JwtConstant {

    // 基本url
    public static final String BASE_DOMAIN_URL = "http://localhost:8080/";

    //jwt密钥，随便设，建议长一点
    public static final String JWT_SECRET = "langfeiyesabcdefghijklmnopqrstuvwxyz11111111111";
    //jwt失效时间，单位秒。这里一共是24小时
    public static final Long JWT_EXPIRATION = 24 * 60 * 60 * 1000L;
    //jwt 创建时间
    public static final String JWT_CREATE_TIME = "jwt_create_time";

    //jwt 用户信息-key
    public static final String USER_INFO_KEY = "user_info_key";
    //jwt 用户信息-id
    public static final String USER_INFO_ID = "user_info_id";
    //jwt 用户信息-username
    public static final String USER_INFO_USERNAME = "user_info_username";
}
```

## JWT工具类

```java
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

/**
 * JwtToken工具类
 */
public class JwtTokenUtil {


    /**
     * 从数据声明生成令牌
     *
     * @param claims 数据声明：自定义载荷
     * @return 令牌
     */
    public static String createToken(Map<String, Object> claims) {
        String token = Jwts.builder()
                //header有默认值，可以不设置
                //.setHeader(new HashMap<>())
                //API给的载荷，按需设置即可
                //.setAudience("Audience")
                //.setIssuer("Issuer")
                //.setSubject("Subject")
                //.setNotBefore(new Date())
                //.setIssuedAt(new Date())
                //.setId("jwt id")
                .setClaims(claims)//把自定义荷载存储到里面
                .setExpiration(generateExpirationDate())//设置失效时间
                .signWith(Keys.hmacShaKeyFor(Decoders.BASE64.decode(JwtConstant.JWT_SECRET))) //签名
                .compact();
        return token;
    }

    /**
     * 从令牌中获取数据声明
     *
     * @param token 令牌
     * @return 数据声明
     */
    public static Claims parseToken(String token){
        Claims claims=null;
        try{
            claims = Jwts.parserBuilder()
                    .setSigningKey(Decoders.BASE64.decode(JwtConstant.JWT_SECRET))
                    .build()
                    .parseClaimsJws(token)
                    .getBody();
        }catch (Exception e){
            e.printStackTrace();
        }
        return claims;
    }

    /**
     * 生成token失效时间
     */
    private static Date generateExpirationDate() {
        //失效时间是当前系统的时间+我们在配置文件里定义的时间
        return new Date(System.currentTimeMillis()+JwtConstant.JWT_EXPIRATION);
    }

    /**
     * 根据token获取用户名
     */
    public static String getUserName(String token){
        Claims claims = parseToken(token);
        return getValue(claims, JwtConstant.USER_INFO_USERNAME);
    }

    /**
     * 验证token是否有效
     */
    public static boolean validateToken(String token){
        //claims 为null 意味着要门jwt被修改
        Claims claims = parseToken(token);
        return claims != null &&!isTokenExpired(token);
    }

    /**
     * 判断token是否已经失效
     * @param token
     * @return
     */
    public static boolean isTokenExpired(String token) {
        //先获取之前设置的token的失效时间
        Date expireDate=getExpiredDate(token);
        return expireDate.before(new Date()); //判断下，当前时间是都已经在expireDate之后
    }

    /**
     * 根据token获取失效时间
     * 也是先从token中获取荷载
     * 然后从荷载中拿到到设置的失效时间
     * @param token
     * @return
     */
    private static Date getExpiredDate(String token) {
        Claims claims=parseToken(token);
        return claims.getExpiration();
    }


    /**
     * 刷新我们的token：重新构建jwt
     */
    public static String refreshToken(String token){
        Claims claims=parseToken(token);
        claims.put(JwtConstant.JWT_CREATE_TIME,new Date());
        return createToken(claims);
    }

    /**
     * 根据身份信息获取键值
     *
     * @param claims 身份信息
     * @param key 键
     * @return 值
     */
    public static String getValue(Claims claims, String key){
        return claims.get(key) != null ? claims.get(key).toString():null;
    }
}
```

# 案例1：邮件激活

**需求：当用户注册成功，给指定邮箱发送一个激活链接，当用户点击激活链接后，激活账号。**

![img/邮箱激活案例.png](https://image.itbaima.net/images/253/image-2023090216526076.png)

**分析**
* 设计一个注册接口/regist，请求成功后模拟下发激活链接(链接本质是激活接口参数为：jwt)
* 设计激活接口/active，接收参数为jwt

***
**代码实现**

先创建项目：mail-active-demo

导入jjwt依赖，以及对应的JWT常量类和JWT工具类

用户注册实体类
```java
public class User {
    private Long id;
    private String username;
    private String password;
    private int state;


    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public int getState() {
        return state;
    }

    public void setState(int state) {
        this.state = state;
    }
}

```



接口：

```java
package com.langfeiyes.mail.controller;

import com.langfeiyes.mail.util.JwtConstant;
import com.langfeiyes.mail.entity.User;
import com.langfeiyes.mail.util.JwtTokenUtil;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.Random;

@RestController
public class UserController {
    //模拟需要缓存的用户库
    //key: url, value: 要激活用户
    private static Map<String, User> map = new HashMap<>();

    @GetMapping("/regist")
    public String regist(User user){
        //假装成功
        System.out.println("注册成功");
        user.setId(new Random().nextLong());
        //创建jwt
        Map<String, Object> claims = new HashMap<>();
        claims.put(JwtConstant.USER_INFO_ID, user.getId());
        claims.put(JwtConstant.USER_INFO_USERNAME, user.getUsername());
        claims.put(JwtConstant.JWT_CREATE_TIME, new Date());
        String jwt = JwtTokenUtil.createToken(claims);

        //缓存jwt
        map.put(jwt, user);
        return JwtConstant.BASE_DOMAIN_URL + "/active?jwt=" + jwt;
    }


    @GetMapping("/active")
    public String active(String jwt){
        User user = map.get(jwt);
        if(user != null && JwtTokenUtil.validateToken(jwt)){
            map.remove(jwt);
            return "执行激活逻辑...";
        }else{
            return "参数不合法...";
        }
    }
}

```

启动项目

```java
package com.langfeiyes.mail;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

测试

浏览器发起2个请求：

注册：http://localhost:8080/regist?username=dafei&password=666

激活：http://localhost:8080//active?jwt=eyJhbGciOiJIUzUxMiJ9.eyJjcmVhdGVfdGltZSI6MTY4MTYxNzA2MDg3NSwiaWQiOjEsInVzZXJuYW1lIjoiZGFmZWkiLCJleHAiOjE2ODE3MDM0NjB9.vQcsXUaEictz3QgjUBKwAV1qlou9yFCSMo4H6OaArz1ReEFzXt6klziHqonvsEfkv9aYdDc6G-vKVO9Zh1kcXw



# 案例2：JWT令牌登录

**需求：设计/login 与 /list 2个接口实现登录与列表逻辑，注意访问/list接口必须进行登录校验**
要求：使用Spring security + JWT

**分析**：
* 设计2个接口，/login登录成功创建JWT响应到客户端
* 设计登录检查拦截器，当访问/list接口时进行登录拦截

**代码设计**：

![img/JWT登录代码设计.png](img/JWT登录代码设计.png)

***
**代码实现**

创建项目：security-jwt-demo

导入jjwt依赖，以及对应的JWT常量类和JWT工具类

实体类User--登录主体

```java
package com.langfeiyes.jwt.entity;

public class User {
    private Long id;
    private String username;
    private String password;
    private int state;

    public User() {
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public int getState() {
        return state;
    }

    public void setState(int state) {
        this.state = state;
    }
}
```

接口响应类-R

```java
package com.langfeiyes.jwt.util;

public class R {
    private int code;
    private String msg;
    private Object data;

    public R() {
    }

    public R(int code, String msg, Object data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }

    public static R ok(){
        return new R(200, "操作成功", null);
    }
    public static R ok(Object data){
        return new R(200, "操作成功", data);
    }
    public static R fail(){
        return new R(500, "操作失败", null);
    }
    public static R fail(Object data){
        return new R(500, "操作失败", data);
    }
    public static R fail(String msg){
        return new R(500, msg, null);
    }

    public int getCode() {
        return code;
    }

    public String getMsg() {
        return msg;
    }

    public Object getData() {
        return data;
    }
}

```

登录过滤器：JwtLoginFilter--做jwt登录

```java
public class JwtLoginFilter extends UsernamePasswordAuthenticationFilter {
    private AuthenticationManager authenticationManager;
    public JwtLoginFilter(AuthenticationManager authenticationManager) {
        this.authenticationManager = authenticationManager;

        this.setRequiresAuthenticationRequestMatcher(new AntPathRequestMatcher("/login", "GET"));
    }



    /**
     * 拦截登录。获取表单的用户名与密码
     */
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        //使用 请求体 传递登录参数，更加安全
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        return authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(username, password));
    }

    /**
     * 登录成功后调用的方法
     * 返回token
     */
    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {

        //security 登录成功封装实体验证
        UserDetails userDetails = (UserDetails) authResult.getPrincipal();
        //根据用户名生成token
        //创建jwt
        Map<String, Object> claims = new HashMap<>();
        claims.put(JwtConstant.USER_INFO_USERNAME, userDetails.getUsername());
        claims.put(JwtConstant.JWT_CREATE_TIME, new Date());
        String jwt = JwtTokenUtil.createToken(claims);

        String token = JwtTokenUtil.createToken(claims);
        response.setHeader("token", token);
        response.setContentType("application/json;charset=utf-8");
        response.getWriter().print(new ObjectMapper().writeValueAsString(R.ok("登录成功")));
    }

    /**
     * 登录失败后调用的方法
     */
    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) throws IOException, ServletException {
        response.setContentType("application/json;charset=utf-8");
        response.getWriter().print(new ObjectMapper().writeValueAsString(R.fail(failed.getMessage())));
    }
}

```

登录拦截过滤器-JwtAuthFilter--登录检查--权限检查

```java
public class JwtAuthFilter extends BasicAuthenticationFilter {
    public JwtAuthFilter(AuthenticationManager authenticationManager) {
        super(authenticationManager);
    }
    /**
     * 对HTTP请求头做处理
     */
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
        //授权
        UsernamePasswordAuthenticationToken authRequest = getAuthentication(request);
        //授权失败
        if (authRequest == null) {
            chain.doFilter(request, response);
            return;
        }
        //如果有授权，放到权限上下文（容器）中
        SecurityContextHolder.getContext().setAuthentication(authRequest);
        chain.doFilter(request, response);
    }

    /**
     * 认证token是否合法，若合法，返回认证，否则返回null
     */
    private UsernamePasswordAuthenticationToken getAuthentication(HttpServletRequest request) {
        //获取token
        String token = request.getHeader("token");

        if (!StringUtils.isEmpty(token) && JwtTokenUtil.validateToken(token)) {
            //从token中获取username
            String username = JwtTokenUtil.getUserName(token);

            return new UsernamePasswordAuthenticationToken(username, token, Arrays.asList(new SimpleGrantedAuthority("admin")));
        }
        return null;
    }
}

```

security整体配置类-JwtWebSecurityConfig

```java
@Configuration
public class JwtWebSecurityConfig extends WebSecurityConfigurerAdapter {



    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("dafei")
                .password("666")
                .roles("admin")
                .and()
                .passwordEncoder(NoOpPasswordEncoder.getInstance());

    }



    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .exceptionHandling()

                .and().authorizeRequests()
                .antMatchers("/login").permitAll()			//所有请求都可以访问
                .antMatchers("/list").authenticated()
                .and().logout()
                .and()
                .addFilter(new JwtLoginFilter(authenticationManager()))	//登录时的过滤器
                .addFilter(new JwtAuthFilter(authenticationManager()))	//验证JWT的过滤器
                .httpBasic();
    }

}
```

访问接口-UserController

```java
package com.langfeiyes.jwt.controller;

import com.langfeiyes.jwt.util.R;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {

    @GetMapping("/list")
    public R list(){
        return R.ok("list....");
    }
}

```

启动项目

```java
package com.langfeiyes.jwt;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}

```

测试

```java
GET http://localhost:8080/login?username=dafei&password=666

<> 2023-04-16T230507.200.json
    
###
GET http://localhost:8080/list
Accept: */*
token: eyJhbGciOiJIUzUxMiJ9.eyJjcmVhdGVfdGltZSI6MTY4MTY1NzUwNzE1NCwidXNlcm5hbWUiOiJkYWZlaSIsImV4cCI6MTY4MTc0MzkwN30.q8X7IisUgF8if299exu1jU-0hgZOFzgUABt9SynqQ2HdyVJJqfAZpywVmyvRLQ8-n5hLf-JtF2mjAbQBlfQZwg

<> 2023-04-16T231623.200.json
```

# JWT避坑指南

## 令牌泄露 

因为JWT是无状态的，当JWT令牌颁发后，在有效时间内，是无法进行销毁，所以就存在很大隐患：**令牌泄露**

**避坑**：颁发JWT令牌时，在Redis中也缓存一份，当判定某个JWT泄露了，立即移除Redis中的JWT。当接口发起请求时，强制用户重新进行身份验证，直至验证成功。
>但这样的话，相当于是有状态的

## 敏感操作 

基于JWT无状态性，同时泄露可能很大，当涉及到敏感数据变动时，应当临时再做检查。

**避坑**：在涉及到诸如新增，修改，删除，上传，下载等敏感性操作时，强制检查用户身份，如手机验证码，扫描二维码等手段，确认操作者是用户本人。



## 超频识别与限制 

当JWT令牌被盗取，一般会出现高频次的系统访问。针对这种情况，监控用户端在单位时间内的请求次数，当单位时间内的请求次数超出预定阈值值，则判定该用户JWT令牌异常。

**避坑**：当判断JWT令牌异常，直接进行限制(比如：IP限流，JWT黑名单等)。



## 地域检查

一般情况下，用户活动范围是固定的，意味着JWT客户端访问IP相对固定。如果JWT泄露，可能会出现异地登录的情况。

**避坑**：对JWT进行异地访问检查，有效时间内，IP频繁变动可判断为JWT泄露。



## 检查客户端

对于APP产品来说，客户端一般是固定的，基本为移动设备，可以结合设备机器码进行绑定。

**避坑**：将JWT与机器码绑定，存储在服务端，当客户端发起请求时，通过检查客户端的机器码与服务端保存的机器码是否匹配，来判断JWT是否泄露。



## 限时，限数，限频

JWT令牌泄露是无法避免，但是可以进行泄露识别，做好泄露后补救保证系统安全。

**避坑**：对客户端进行合理限制，比如限制每个客户端的 JWT 令牌数量、访问频率、JWT令牌时效等，以降低 JWT 令牌泄露的风险。

# 回顾

**问：什么是JWT？解释一下它的结构。**

JWT是一种开放标准，用于在网络上安全地传输信息。它由三部分组成：头部、载荷和签名。头部包含令牌的元数据，载荷包含实际的信息（例如用户ID、角色等），签名用于验证令牌是否被篡改。

**问：JWT的优点是什么？它与传统的session-based身份验证相比有什么优缺点？**

JWT的优点包括无状态、可扩展、跨语言、易于实现和良好的安全性。相比之下，传统的session-based身份验证需要在服务端维护会话状态，使得服务端的负载更高，并且不适用于分布式系统。

**问：在JWT的结构中，分别有哪些部分？每个部分的作用是什么？**

JWT的结构由三部分组成：头部、载荷和签名。头部包含令牌类型和算法，载荷包含实际的信息，签名由头部、载荷和密钥生成。

**问：JWT如何工作？从开始到验证过程的完整流程是怎样的？**

JWT的工作流程分为三个步骤：生成令牌、发送令牌、验证令牌。在生成令牌时，服务端使用密钥对头部和载荷进行签名。在发送令牌时，将令牌发送给客户端。在验证令牌时，客户端从令牌中解析出头部和载荷，并使用相同的密钥验证签名。

**问：什么是JWT的签名？为什么需要对JWT进行签名？如何验证JWT的签名？**

JWT的签名是由头部、载荷和密钥生成的，用于验证令牌是否被篡改。签名使用HMAC算法或RSA算法生成。在验证JWT的签名时，客户端使用相同的密钥和算法生成签名，并将生成的签名与令牌中的签名进行比较。

**问：什么是JWT的令牌刷新？为什么需要这个功能？**

令牌刷新是一种机制，用于解决JWT过期后需要重新登录的问题。在令牌刷新中，服务端生成新的JWT，并将其发送给客户端。客户端使用新的JWT替换旧的JWT，从而延长令牌的有效期。

**问：JWT是否加密？如果是，加密的部分是哪些？如果不是，那么它如何保证数据安全性？**

JWT本身并不加密，但可以在载荷中包含敏感信息。为了保护这些信息，可以使用JWE（JSON Web Encryption）对载荷进行加密。如果不加密，则需要在生成JWT时确保不在载荷中包含敏感信息。

**问：在JWT中，如何处理Token过期的问题？有哪些方法可以处理？**

JWT过期后，客户端需要重新获取新的JWT。可以通过在JWT中包含过期时间或使用refresh token等机制来解决过期问题。

**问：JWT和OAuth2有什么关系？它们之间有什么区别？**

JWT和OAuth2都是用于身份验证和授权的开放标准。JWT是一种身份验证机制，而OAuth2是一种授权机制。JWT用于在不同的系统中安全地传输信息，OAuth2用于授权第三方应用程序访问受保护的资源。

**问：JWT在什么场景下使用较为合适？它的局限性是什么？**

JWT在单体应用或微服务架构中的使用比较合适。它的局限性包括无法撤销、令牌较大、无法处理并发等问题。在需要针对每次请求进行访问控制或需要撤销令牌的情况下，JWT可能不是最佳选择。