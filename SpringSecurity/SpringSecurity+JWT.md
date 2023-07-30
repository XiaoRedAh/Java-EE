**SpringSecurity + JWT方案**

>针对SpringBoot2.x+SpringSecurity5.x版本

SpringSecurity是一个基于Spring开发的非常强大的权限验证框架，其核心功能包括：

- 认证 （用户登录）
- 授权 （此用户能够做哪些事情）
- 攻击防护 （防止伪造身份攻击）

相比另外一个安全框架Shiro，SpringSecurity提供了更丰富的功能，社区资源也比Shiro丰富。

​一般来说中大型的项目都是使用SpringSecurity来做安全框架，小项目用Shiro的比较多

# 简单搭建

## 准备工作

先搭建一个简单的SpringBoot工程

① 设置父工程 添加依赖

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.0</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
```

② 创建启动类

```java
@SpringBootApplication
public class SecurityApplication {

    public static void main(String[] args) {
        SpringApplication.run(SecurityApplication.class,args);
    }
}
```

③ 创建Controller

```java
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String hello(){
        return "hello";
    }
}
```

## 引入SpringSecurity

​在SpringBoot项目中使用SpringSecurity只需要引入依赖即可实现

入门案例。

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
```

​引入依赖后在尝试去访问之前的接口就会自动跳转到一个SpringSecurity的默认登陆页面，默认用户名是user,密码会输出在控制台。

​必须登陆之后才能对接口进行访问。

# SpringSecurity完整流程

​SpringSecurity的原理其实就是一个过滤器链，内部包含了提供各种功能的过滤器。

​图中只展示了核心过滤器，其它的非核心过滤器并没有在图中展示。

![image-20211214144425527](https://image.itbaima.net/images/253/image-20230730156375800.png)

**UsernamePasswordAuthenticationFilter（负责认证）**：负责处理在登录页面填写用户名密码后的登录请求

**ExceptionTranslationFilter**：处理过滤器链中抛出的任何AccessDeniedException和AuthenticationException

**FilterSecurityInterceptor（负责授权）**：负责权限校验的过滤器

可以通过Debug查看当前系统中SpringSecurity过滤器链中有哪些过滤器及它们的顺序。

![image-20211214145824903](https://image.itbaima.net/images/253/image-20230730153937003.png)

# 认证

## 登录校验流程

前后端分离项目中的登录校验流程：

核心就是token（理解为加密的字符串）

![image-20211215094003288](https://image.itbaima.net/images/253/image-20230730154060669.png)

## 认证流程详解

想要知道如何实现自己的登录流程，就必须要先了解SpringSecurity提供的默认流程。

自定义的登录流程在SpringSecurity提供的默认流程上做出相应修改即可

SpringSecurity默认的认证流程

*注意：默认的UserDetailsService接口的实现类是inMemoryUserDetailsManager,它是从内存中查。我们期望用数据库去查，那么就要自己去实现UserDetailsService*

![image-20211214151515385](https://image.itbaima.net/images/253/image-20230730159676361.png)

Authentication接口: 它的实现类，表示当前访问系统的用户，封装了用户相关信息。

AuthenticationManager接口：定义了认证Authentication的方法 

UserDetailsService接口：加载用户特定数据的核心接口。里面定义了一个根据用户名查询用户信息的方法。

UserDetails接口：提供核心用户信息。通过UserDetailsService根据用户名获取处理的用户信息要封装成UserDetails对象返回。然后将这些信息封装到Authentication对象中。

## 认证方案一

## 思路分析（重点）

![img/image-20211215095331510.png](https://image.itbaima.net/images/253/image-20230730157782626.png)

*登录*

​①自定义登录接口controller  

绕过UsernamePasswordAuthenticationFilter，将前端提交的用户名和密码给到controller，在controller中​调用ProviderManager的方法进行认证。如果认证通过生成jwt，然后用userid作为key，用户信息作为value存入redis。（在校验的时候就可以从redis中拿，减小数据库的压力）

​②自定义UserDetailsService 

实现loadUserByUsername方法，方法里要做：
  1. 从数据库查询用户信息（登录功能）
  2. 如果查到，就再去查询对应的权限信息（授权功能）
  3. 封装成UserDetails对象返回（这要求再去创建一个UserDetails的实现类）

*校验*

​①定义Jwt认证过滤器
  1. ​获取token
  2. ​解析token获取其中的userid
  3. ​从redis中获取用户信息
  4. 封装Authentication对象​存入SecurityContextHolder

## 准备工作

①添加依赖

```xml
        <!--redis依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!--fastjson依赖-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.33</version>
        </dependency>
        <!--jwt依赖-->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.0</version>
        </dependency>
```

② 添加Redis相关配置

com.sangeng.utils.FastJsonRedisSerializer

```java
/**
 * Redis使用FastJson序列化
 */
public class FastJsonRedisSerializer<T> implements RedisSerializer<T>
{

    public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");

    private Class<T> clazz;

    static
    {
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
    }

    public FastJsonRedisSerializer(Class<T> clazz)
    {
        super();
        this.clazz = clazz;
    }

    @Override
    public byte[] serialize(T t) throws SerializationException
    {
        if (t == null)
        {
            return new byte[0];
        }
        return JSON.toJSONString(t, SerializerFeature.WriteClassName).getBytes(DEFAULT_CHARSET);
    }

    @Override
    public T deserialize(byte[] bytes) throws SerializationException
    {
        if (bytes == null || bytes.length <= 0)
        {
            return null;
        }
        String str = new String(bytes, DEFAULT_CHARSET);

        return JSON.parseObject(str, clazz);
    }


    protected JavaType getJavaType(Class<?> clazz)
    {
        return TypeFactory.defaultInstance().constructType(clazz);
    }
}
```

com.sangeng.config.RedisConfig

```java
@Configuration
public class RedisConfig {

    @Bean
    @SuppressWarnings(value = { "unchecked", "rawtypes" })
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory connectionFactory)
    {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);

        FastJsonRedisSerializer serializer = new FastJsonRedisSerializer(Object.class);

        // 使用StringRedisSerializer来序列化和反序列化redis的key值
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(serializer);

        // Hash的key也采用StringRedisSerializer的序列化方式
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(serializer);

        template.afterPropertiesSet();
        return template;
    }
}
```

③ 响应类

com.sangeng.domin.ResponseResult

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
@Data
public class ResponseResult<T> {
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

④工具类

com.sangeng.utils.JwtUtil

```java
/**
 * JWT工具类
 */
public class JwtUtil {

    //有效期为
    public static final Long JWT_TTL = 60 * 60 *1000L;// 60 * 60 *1000  一个小时
    //设置秘钥明文
    public static final String JWT_KEY = "sangeng";

    public static String getUUID(){
        String token = UUID.randomUUID().toString().replaceAll("-", "");
        return token;
    }
    
    /**
     * 生成jtw
     * @param subject token中要存放的数据（json格式）
     * @return
     */
    public static String createJWT(String subject) {
        JwtBuilder builder = getJwtBuilder(subject, null, getUUID());// 设置过期时间
        return builder.compact();
    }

    /**
     * 生成jtw
     * @param subject token中要存放的数据（json格式）
     * @param ttlMillis token超时时间
     * @return
     */
    public static String createJWT(String subject, Long ttlMillis) {
        JwtBuilder builder = getJwtBuilder(subject, ttlMillis, getUUID());// 设置过期时间
        return builder.compact();
    }

    private static JwtBuilder getJwtBuilder(String subject, Long ttlMillis, String uuid) {
        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
        SecretKey secretKey = generalKey();
        long nowMillis = System.currentTimeMillis();
        Date now = new Date(nowMillis);
        if(ttlMillis==null){
            ttlMillis=JwtUtil.JWT_TTL;
        }
        long expMillis = nowMillis + ttlMillis;
        Date expDate = new Date(expMillis);
        return Jwts.builder()
                .setId(uuid)              //唯一的ID
                .setSubject(subject)   // 主题  可以是JSON数据
                .setIssuer("sg")     // 签发者
                .setIssuedAt(now)      // 签发时间
                .signWith(signatureAlgorithm, secretKey) //使用HS256对称加密算法签名, 第二个参数为秘钥
                .setExpiration(expDate);
    }

    /**
     * 创建token
     * @param id
     * @param subject
     * @param ttlMillis
     * @return
     */
    public static String createJWT(String id, String subject, Long ttlMillis) {
        JwtBuilder builder = getJwtBuilder(subject, ttlMillis, id);// 设置过期时间
        return builder.compact();
    }

    public static void main(String[] args) throws Exception {
        String token = "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiJjYWM2ZDVhZi1mNjVlLTQ0MDAtYjcxMi0zYWEwOGIyOTIwYjQiLCJzdWIiOiJzZyIsImlzcyI6InNnIiwiaWF0IjoxNjM4MTA2NzEyLCJleHAiOjE2MzgxMTAzMTJ9.JVsSbkP94wuczb4QryQbAke3ysBDIL5ou8fWsbt_ebg";
        Claims claims = parseJWT(token);
        System.out.println(claims);
    }

    /**
     * 生成加密后的秘钥 secretKey
     * @return
     */
    public static SecretKey generalKey() {
        byte[] encodedKey = Base64.getDecoder().decode(JwtUtil.JWT_KEY);
        SecretKey key = new SecretKeySpec(encodedKey, 0, encodedKey.length, "AES");
        return key;
    }
    
    /**
     * 解析
     *
     * @param jwt
     * @return
     * @throws Exception
     */
    public static Claims parseJWT(String jwt) throws Exception {
        SecretKey secretKey = generalKey();
        return Jwts.parser()
                .setSigningKey(secretKey)
                .parseClaimsJws(jwt)
                .getBody();
    }
}
```

com.sangeng.utils.RedisCache

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.BoundSetOperations;
import org.springframework.data.redis.core.HashOperations;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.stereotype.Component;

import java.util.*;
import java.util.concurrent.TimeUnit;

@SuppressWarnings(value = { "unchecked", "rawtypes" })
@Component
public class RedisCache
{
    @Autowired
    public RedisTemplate redisTemplate;

    /**
     * 缓存基本的对象，Integer、String、实体类等
     *
     * @param key 缓存的键值
     * @param value 缓存的值
     */
    public <T> void setCacheObject(final String key, final T value)
    {
        redisTemplate.opsForValue().set(key, value);
    }

    /**
     * 缓存基本的对象，Integer、String、实体类等
     *
     * @param key 缓存的键值
     * @param value 缓存的值
     * @param timeout 时间
     * @param timeUnit 时间颗粒度
     */
    public <T> void setCacheObject(final String key, final T value, final Integer timeout, final TimeUnit timeUnit)
    {
        redisTemplate.opsForValue().set(key, value, timeout, timeUnit);
    }

    /**
     * 设置有效时间
     *
     * @param key Redis键
     * @param timeout 超时时间
     * @return true=设置成功；false=设置失败
     */
    public boolean expire(final String key, final long timeout)
    {
        return expire(key, timeout, TimeUnit.SECONDS);
    }

    /**
     * 设置有效时间
     *
     * @param key Redis键
     * @param timeout 超时时间
     * @param unit 时间单位
     * @return true=设置成功；false=设置失败
     */
    public boolean expire(final String key, final long timeout, final TimeUnit unit)
    {
        return redisTemplate.expire(key, timeout, unit);
    }

    /**
     * 获得缓存的基本对象。
     *
     * @param key 缓存键值
     * @return 缓存键值对应的数据
     */
    public <T> T getCacheObject(final String key)
    {
        ValueOperations<String, T> operation = redisTemplate.opsForValue();
        return operation.get(key);
    }

    /**
     * 删除单个对象
     *
     * @param key
     */
    public boolean deleteObject(final String key)
    {
        return redisTemplate.delete(key);
    }

    /**
     * 删除集合对象
     *
     * @param collection 多个对象
     * @return
     */
    public long deleteObject(final Collection collection)
    {
        return redisTemplate.delete(collection);
    }

    /**
     * 缓存List数据
     *
     * @param key 缓存的键值
     * @param dataList 待缓存的List数据
     * @return 缓存的对象
     */
    public <T> long setCacheList(final String key, final List<T> dataList)
    {
        Long count = redisTemplate.opsForList().rightPushAll(key, dataList);
        return count == null ? 0 : count;
    }

    /**
     * 获得缓存的list对象
     *
     * @param key 缓存的键值
     * @return 缓存键值对应的数据
     */
    public <T> List<T> getCacheList(final String key)
    {
        return redisTemplate.opsForList().range(key, 0, -1);
    }

    /**
     * 缓存Set
     *
     * @param key 缓存键值
     * @param dataSet 缓存的数据
     * @return 缓存数据的对象
     */
    public <T> BoundSetOperations<String, T> setCacheSet(final String key, final Set<T> dataSet)
    {
        BoundSetOperations<String, T> setOperation = redisTemplate.boundSetOps(key);
        Iterator<T> it = dataSet.iterator();
        while (it.hasNext())
        {
            setOperation.add(it.next());
        }
        return setOperation;
    }

    /**
     * 获得缓存的set
     *
     * @param key
     * @return
     */
    public <T> Set<T> getCacheSet(final String key)
    {
        return redisTemplate.opsForSet().members(key);
    }

    /**
     * 缓存Map
     *
     * @param key
     * @param dataMap
     */
    public <T> void setCacheMap(final String key, final Map<String, T> dataMap)
    {
        if (dataMap != null) {
            redisTemplate.opsForHash().putAll(key, dataMap);
        }
    }

    /**
     * 获得缓存的Map
     *
     * @param key
     * @return
     */
    public <T> Map<String, T> getCacheMap(final String key)
    {
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * 往Hash中存入数据
     *
     * @param key Redis键
     * @param hKey Hash键
     * @param value 值
     */
    public <T> void setCacheMapValue(final String key, final String hKey, final T value)
    {
        redisTemplate.opsForHash().put(key, hKey, value);
    }

    /**
     * 获取Hash中的数据
     *
     * @param key Redis键
     * @param hKey Hash键
     * @return Hash中的对象
     */
    public <T> T getCacheMapValue(final String key, final String hKey)
    {
        HashOperations<String, String, T> opsForHash = redisTemplate.opsForHash();
        return opsForHash.get(key, hKey);
    }

    /**
     * 删除Hash中的数据
     * 
     * @param key
     * @param hkey
     */
    public void delCacheMapValue(final String key, final String hkey)
    {
        HashOperations hashOperations = redisTemplate.opsForHash();
        hashOperations.delete(key, hkey);
    }

    /**
     * 获取多个Hash中的数据
     *
     * @param key Redis键
     * @param hKeys Hash键集合
     * @return Hash对象集合
     */
    public <T> List<T> getMultiCacheMapValue(final String key, final Collection<Object> hKeys)
    {
        return redisTemplate.opsForHash().multiGet(key, hKeys);
    }

    /**
     * 获得缓存的基本对象列表
     *
     * @param pattern 字符串前缀
     * @return 对象列表
     */
    public Collection<String> keys(final String pattern)
    {
        return redisTemplate.keys(pattern);
    }
}
```

com.sangeng.utils.WebUtils

```java
public class WebUtils
{
    /**
     * 将字符串渲染到客户端
     * 
     * @param response 渲染对象
     * @param string 待渲染的字符串
     * @return null
     */
    public static String renderString(HttpServletResponse response, String string) {
        try
        {
            response.setStatus(200);
            response.setContentType("application/json");
            response.setCharacterEncoding("utf-8");
            response.getWriter().print(string);
        }
        catch (IOException e)
        {
            e.printStackTrace();
        }
        return null;
    }
}
```

⑤实体类

com.sangeng.domain.User

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User implements Serializable {
    private static final long serialVersionUID = -40356785423868312L;
    
    /**
    * 主键
    */
    private Long id;
    /**
    * 用户名
    */
    private String userName;
    /**
    * 昵称
    */
    private String nickName;
    /**
    * 密码
    */
    private String password;
    /**
    * 账号状态（0正常 1停用）
    */
    private String status;
    /**
    * 邮箱
    */
    private String email;
    /**
    * 手机号
    */
    private String phonenumber;
    /**
    * 用户性别（0男，1女，2未知）
    */
    private String sex;
    /**
    * 头像
    */
    private String avatar;
    /**
    * 用户类型（0管理员，1普通用户）
    */
    private String userType;
    /**
    * 创建人的用户id
    */
    private Long createBy;
    /**
    * 创建时间
    */
    private Date createTime;
    /**
    * 更新人
    */
    private Long updateBy;
    /**
    * 更新时间
    */
    private Date updateTime;
    /**
    * 删除标志（0代表未删除，1代表已删除）
    */
    private Integer delFlag;
}
```

## 实现数据库校验用户

​通过之前的分析，可以自定义一个UserDetailsService,让SpringSecurity使用我们的UserDetailsService。

我们自己的UserDetailsService从数据库中查询用户名和密码。

**准备工作**

创建一个用户表， 建表语句如下：

```mysql
CREATE TABLE `sys_user` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `user_name` VARCHAR(64) NOT NULL DEFAULT 'NULL' COMMENT '用户名',
  `nick_name` VARCHAR(64) NOT NULL DEFAULT 'NULL' COMMENT '昵称',
  `password` VARCHAR(64) NOT NULL DEFAULT 'NULL' COMMENT '密码',
  `status` CHAR(1) DEFAULT '0' COMMENT '账号状态（0正常 1停用）',
  `email` VARCHAR(64) DEFAULT NULL COMMENT '邮箱',
  `phonenumber` VARCHAR(32) DEFAULT NULL COMMENT '手机号',
  `sex` CHAR(1) DEFAULT NULL COMMENT '用户性别（0男，1女，2未知）',
  `avatar` VARCHAR(128) DEFAULT NULL COMMENT '头像',
  `user_type` CHAR(1) NOT NULL DEFAULT '1' COMMENT '用户类型（0管理员，1普通用户）',
  `create_by` BIGINT(20) DEFAULT NULL COMMENT '创建人的用户id',
  `create_time` DATETIME DEFAULT NULL COMMENT '创建时间',
  `update_by` BIGINT(20) DEFAULT NULL COMMENT '更新人',
  `update_time` DATETIME DEFAULT NULL COMMENT '更新时间',
  `del_flag` INT(11) DEFAULT '0' COMMENT '删除标志（0代表未删除，1代表已删除）',
  PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COMMENT='用户表'
```

​引入MybatisPuls和mysql驱动的依赖

```xml
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.3</version>
        </dependency>
        <!--mysql驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.27</version>
        </dependency>
```

​配置数据库信息

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mybatis_db?characterEncoding=utf-8&serverTimezone=UTC
    username: root
    password: cronaldo7
    driver-class-name: com.mysql.cj.jdbc.Driver
```

​定义Mapper接口

com.sangeng.mapper.UserMapper

```java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.sangeng.domin.User;
@Repository 
public interface UserMapper extends BaseMapper<User> {
}
```

​修改User实体类

```java
类名上加@TableName(value = "sys_user") ,id字段上加 @TableId
```

​配置Mapper扫描

```java
@SpringBootApplication
@MapperScan("com.sangeng.mapper")
public class SecurityApplication {

    public static void main(String[] args) {
        SpringApplication.run(SecurityApplication.class,args);
    }
}
```

​添加junit依赖

```java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
```

​测试mapper是否能正常使用

test.java.com.sangeng.MapperTest

```java
import com.sangeng.domin.User;
import com.sangeng.mapper.UserMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;
@SpringBootTest
public class MapperTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testUserMapper(){
        List<User> users = userMapper.selectList(null);
        System.out.println(users);
    }
}
```

### 核心代码实现

①创建一个类实现UserDetailsService接口，重写其中的方法。

现在只是在做登录功能：根据用户名从数据库中查询用户信息

授权功能暂时放着

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private UserMapper userMapper;

    //将登录时输入的用户名传入这个方法中进行查询用户
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        //从数据库查询用户信息
        LambdaQueryWrapper<User> queryWrapper=new LambdaQueryWrapper<>();
        queryWrapper.eq(User::getUserName,username);
        User user=userMapper.selectOne(queryWrapper);
        //如果没有查到，就抛出异常（安全性校验）
        if(Objects.isNull(user)){
            throw new RuntimeException("用户名或密码错误");
        }
        //TODO 查询对应的权限信息（授权功能）

        //封装成UserDetails对象返回
        return new LoginUser(user);
    }
}        
```

②创建一个实现UserDetails的实体类

因为UserDetailsService方法的返回值是UserDetails类型，所以需要定义一个类，实现该接口，把查到的user用户信息封装在其中（这也要求这个类要有个User属性）

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class LoginUser implements UserDetails {

    private User user;


    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return null;//暂时还没有权限信息，就先返回null就行
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUserName();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

如果现在直接登录，不能成功，显示

```
There is no PasswordEncoder mapped for the id "null"
```

因为现在数据库里的密码是明文存储的，如果想让用户的密码是明文存储，需要在密码前加{noop}。例如

![image-20211216123945882](https://image.itbaima.net/images/253/image-20230730157298191.png)

这样登录的时候就可以用sg作为用户名，1234作为密码来登录了。

### 密码加密存储

​实际项目中不会把密码明文存储在数据库。

​默认使用的PasswordEncoder要求数据库中的密码格式为：{id}password。它会根据id去判断密码的加密方式。

但是一般不会采用这种方式。所以就需要替换PasswordEncoder。

一般使用SpringSecurity提供的**BCryptPasswordEncoder**

只需要把BCryptPasswordEncoder对象注入Spring容器中，SpringSecurity就会使用该PasswordEncoder来进行密码校验。

**定义一个SpringSecurity的配置类**

SpringSecurity要求这个配置类要继承WebSecurityConfigurerAdapter

*注：配置完之后，表的密码就不能明文存储了，必须密文存储才能正确校验*

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    //创建一个BCryptPasswordEncoder注入容器
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

}
```

**测试一下**

```java
public void TestBCryptPasswordEncoder(){
        BCryptPasswordEncoder passwordEncoder=new BCryptPasswordEncoder();
        //加密获得密文
        String str1= passwordEncoder.encode("1234");
        System.out.println(str1);
        //$2a$10$ZBisA5QMj37XY.E0bxM/Gu2.Ajx.I108p.e11itHLWa/B5WcLFh/i
        //将明文与密文比较进行校验
        System.out.println(passwordEncoder.matches("1234",
                "$2a$10$ZBisA5QMj37XY.E0bxM/Gu2.Ajx.I108p.e11itHLWa/B5WcLFh/i"));

    }
```

### 登录接口

让SpringSecurity对自定义的登录接口放行,从而实现用户访问这个接口的时候不用登录也能访问。

​接口中通过AuthenticationManager的authenticate方法来进行用户认证,所以需要在SecurityConfig中配置把AuthenticationManager注入容器。

认证成功的话要生成一个jwt，放入响应中返回。并且为了让用户下回请求时能通过jwt识别出具体的是哪个用户，需要把用户信息存入redis，可以把用户id作为key。

LoginController

```java
@RestController
public class LoginController {

    @Autowired
    private LoginServcie loginServcie;

    @PostMapping("/user/login")
    //从请求体中拿到User
    public ResponseResult login(@RequestBody User user){
        return loginServcie.login(user);
    }
}
```

SecurityConfig

 **ctrl+o看看有什么方法可以重写**

 1. 为authenticationManagerBean()添加@Bean，即可把AuthenticationManager注入到容器中，这样LoginServiceImpl就可以注入使用它了
 2. 重写configure(HttpSecurity http)进行配置



```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {


    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //关闭csrf
                .csrf().disable()
                //不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                // 对于登录接口 允许匿名访问
                .antMatchers("/user/login").anonymous()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
}
```


LoginServiceImpl

 1. 注入AuthenticationManager
 2. authenticate()方法的参数要求是Authentication类型，用ctrl+alt看这个接口有什么实现类。找到UsernamePasswordAuthenticationToken这个实现类，使用它来当方法参数（从之前那个流程图里可以知道，要将用户名和密码封装到Authentication对象里）

```java
@Service
public class LoginServiceImpl implements LoginServcie {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private RedisCache redisCache;

    @Override
    public ResponseResult login(User user) {
        //AuthenticationManager的authenticate进行用户认证
        UsernamePasswordAuthenticationToken authenticationToken=new UsernamePasswordAuthenticationToken(user.getUserName(),
        user.getPassword());
        Authentication authenticate= authenticationManager .authenticate(authenticationToken);
        //如果认证没通过，给出提示
        if (Objects.isNull(authenticate)){
            throw new RuntimeException("登录失败");
        }
        //如果认证通过，使用userid生成一个jwt，jwt存入ResponseResult返回
        LoginUser loginUser=(LoginUser) authenticate.getPrincipal();
        String userid = loginUser.getUser().getId().toString();
        String jwt=JwtUtil.createJWT(userid);
        Map<String,String> map=new HashMap<>();
        map.put("token",jwt);
        //把完整的用户信息存入Redis，userid作为key
        redisCache.setCacheObject("login:"+userid,loginUser);
        return new ResponseResult(200,"登录成功",map);
    }
}
```

### 认证过滤器

需要自定义一个过滤器

 1. 获取请求头中的token
 2. 对token进行解析取出其中的userid
 3. 使用userid去redis中获取对应的LoginUser对象
 4. ​然后封装成Authentication对象存入SecurityContextHolder


token认证过滤器代码：

```java
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {

    @Autowired
    private RedisCache redisCache;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        //从请求头获取token
        String token = request.getHeader("token");
        if (!StringUtils.hasText(token)) {
            //没有token，不用解析，直接放行（因为后面还有一系列过滤器对其进行认证分析）
            filterChain.doFilter(request, response);
            return;
        }
        //token不为空，解析token
        String userid;
        try {
            Claims claims = JwtUtil.parseJWT(token);
            userid = claims.getSubject();//拿到userid
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException("token非法");
        }
        //从redis中获取用户信息
        String redisKey = "login:" + userid;
        LoginUser loginUser = redisCache.getCacheObject(redisKey);//根据key从redis中拿到用户
        //判断一下获取的用户为null的情况
        if(Objects.isNull(loginUser)){
            throw new RuntimeException("用户未登录");
        }
        //存入SecurityContextHolder
        //TODO 获取权限信息封装到Authentication中
        UsernamePasswordAuthenticationToken authenticationToken =
                new UsernamePasswordAuthenticationToken(loginUser,null,null);//选用3个参数的这个，因为它会自动设置把第一个参数设置为已认证状态
        SecurityContextHolder.getContext().setAuthentication(authenticationToken);
        //放行
        filterChain.doFilter(request, response);
    }
}
```

SecurityConfig配置认证过滤器：


 1. 注入这个过滤器
 2. addFilterBefore()方法将认证过滤器添加到UsernamePasswordAuthenticationFilter前面

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {


    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }


    @Autowired
    JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //关闭csrf
                .csrf().disable()
                //不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                // 对于登录接口 允许匿名访问
                .antMatchers("/user/login").anonymous()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();

        //把token校验过滤器添加到过滤器链中
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
}
```

### 退出登录

1. 定义一个退出登录接口
2. 获取SecurityContextHolder中的认证信息，通过它来得到userid，通过userid来删除redis中对应的数据。

```java
 @RequestMapping("/user/logout")
    public ResponseResult logout(){
        return loginService.logout();
    }
```

```java
@Service
public class LoginServiceImpl implements LoginServcie {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private RedisCache redisCache;

    @Override
    public ResponseResult login(User user) {
        //AuthenticationManager的authenticate进行用户认证
        UsernamePasswordAuthenticationToken authenticationToken=new UsernamePasswordAuthenticationToken(user.getUserName(),
        user.getPassword());
        Authentication authenticate= authenticationManager .authenticate(authenticationToken);
        //如果认证没通过，给出提示
        if (Objects.isNull(authenticate)){
            throw new RuntimeException("登录失败");
        }
        //如果认证通过，使用userid生成一个jwt，jwt存入ResponseResult返回
        LoginUser loginUser=(LoginUser) authenticate.getPrincipal();
        String userid = loginUser.getUser().getId().toString();
        String jwt=JwtUtil.createJWT(userid);
        Map<String,String> map=new HashMap<>();
        map.put("token",jwt);
        //把完整的用户信息存入Redis，userid作为key
        redisCache.setCacheObject("login:"+userid,loginUser);
        return new ResponseResult(200,"登录成功",map);
    }

    @Override
    public ResponseResult logout() {
        //获取SecurityContextHolder中的用户id
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        LoginUser loginUser = (LoginUser) authentication.getPrincipal();
        Long userid = loginUser.getUser().getId();
        //通过id删除redis中的值
        redisCache.deleteObject("login:"+userid);
        return new ResponseResult(200,"退出成功");
    }
}
```

## configure(HttpSecurity http) 详解

```java
@Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //关闭csrf
                .csrf().disable()
                //不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                // 对于登录接口 允许匿名访问
                .antMatchers("/user/login").anonymous()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();

        //把token校验过滤器添加到过滤器链中
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
    }
```

1. 调用http的方法，每个方法相当于一种配置：http.配置什么.配置细节, 支持链式编程
2. 不同配置之间用.and()隔开
3. http.authorizeRequests()可以对请求认证规则进行配置
   * .antMatchers()指定对某一请求方式/某一请求地址进行配置
     .anyRequest()对任意请求进行统一配置（除了上面特别配置的之外）
     * .anonymous()表示匿名访问：未登录可以访问，已登录不能访问
     * .authenticated()表示需要登录才能访问
     * .permmitAll()表示无论登录还是未登录都能访问

## 其他认证方案

![image-20211214151515385](https://image.itbaima.net/images/253/image-20230730159676361.png)

### 回顾方案1

**这种方案后期拓展（比如验证码校验）会比较方便（在Controller层里拓展，而不是像方案三那样添加过滤器）**

*登录*

​①自定义登录接口controller  

绕过UsernamePasswordAuthenticationFilter，将前端提交的用户名和密码给到controller，在controller中​调用ProviderManager的方法进行认证。如果认证通过生成jwt，然后用userid作为key，用户信息作为value存入redis。（在校验的时候就可以从redis中拿，减小数据库的压力）

​②自定义UserDetailsService 

实现loadUserByUsername方法，方法里要做：
  1. 从数据库查询用户信息（登录功能）
  2. 如果查到，就再去查询对应的权限信息（授权功能）
  3. 封装成UserDetails对象返回（这要求再去创建一个UserDetails的实现类）

*校验*

​①定义Jwt认证过滤器
  1. ​获取token
  2. ​解析token获取其中的userid
  3. ​从redis中获取用户信息
  4. 封装Authentication对象​存入SecurityContextHolder

### 方案2

自定义UsernamePasswordAuthenticationFilter，实现类似功能

### 方案3（另一篇笔记）

**前后端分离项目模板用的就是方案3+Session的认证校验方式，可以去那篇笔记里看具体的使用**

依然使用UsernamePasswordAuthenticationFilter（进行表单登录校验），按默认流程走，直到UserDetailsService这里，自己去实现UserDetailsService（进行数据库校验）。

查完之后，当返回到UsernamePasswordAuthenticationFilter时，使用认证成功处理器，在里面生成JWT以及对应的用户存到Redis里

因为UsernamePasswordAuthenticationFilter仅仅做表单校验（用户名，密码），因此如果还要校验验证码的话，需要在它前面再加一个过滤器，在那个过滤器里校验验证码

#### 认证成功处理器

**注意：如果使用认证成功处理器，那其实就是另一种登录思路和实现**

什么都没配置的情况下，会有UsernamePasswordAuthenticationFilter。
UsernamePasswordAuthenticationFilter使用的是表单验证进行登录的，通过这个过滤器登录成功后，会调用AuthenticationSuccessHandler（认证成功处理器）的onAuthenticationSuccess方法进行认证成功后的处理。

但是方案一的思路其实是绕过了UsernamePasswordAuthenticationFilter，那么也就不存在登录成功后调用AuthenticationSuccessHandler的情况


**自定义成功处理器进行登录成功后的相应处理**

```java
@Component
public class SGSuccessHandler implements AuthenticationSuccessHandler {

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        System.out.println("认证成功了");
    }
}
```

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private AuthenticationSuccessHandler successHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin().successHandler(successHandler);

        http.authorizeRequests().anyRequest().authenticated();
    }
}
```

#### 认证失败处理器

**注意：如果使用认证失败处理器，那其实就是另一种登录思路和实现**

什么都没配置的情况下，会有UsernamePasswordAuthenticationFilter。
UsernamePasswordAuthenticationFilter使用的是表单验证进行登录的，通过这个过滤器认证失败后，会调用AuthenticationFailureHandler（认证失败处理器）的 onAuthenticationFailure方法进行认证失败后的处理。

但是方案一的登录认证思路其实是绕过了UsernamePasswordAuthenticationFilter，那么也就不存在登录失败后调用AuthenticationFailureHandler的情况

**自定义失败处理器进行失败后的相应处理**

```java
@Component
public class SGFailureHandler implements AuthenticationFailureHandler {
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        System.out.println("认证失败了");
    }
}
```

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private AuthenticationSuccessHandler successHandler;

    @Autowired
    private AuthenticationFailureHandler failureHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
                //配置认证成功处理器
                .successHandler(successHandler)
                //配置认证失败处理器
                .failureHandler(failureHandler);

        http.authorizeRequests().anyRequest().authenticated();
    }
}
```

#### 注销成功处理器

方案一的登录认证思路其实是绕过了UsernamePasswordAuthenticationFilter，那么就不存在调用LogoutSuccessHandler的情况

```java
@Component
public class SGLogoutSuccessHandler implements LogoutSuccessHandler {
    @Override
    public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        System.out.println("注销成功");
    }
}
```

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private AuthenticationSuccessHandler successHandler;

    @Autowired
    private AuthenticationFailureHandler failureHandler;

    @Autowired
    private LogoutSuccessHandler logoutSuccessHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
                //配置认证成功处理器
                .successHandler(successHandler)
                //配置认证失败处理器
                .failureHandler(failureHandler);

        http.logout()
                //配置注销成功处理器
                .logoutSuccessHandler(logoutSuccessHandler);

        http.authorizeRequests().anyRequest().authenticated();
    }
}
```

## 更多

理解透上面那个图，其实对哪一个结点进行一些自定义设计都可以实现认证功能

# 授权

权限系统的作用：**不同的用户可以使用不同的功能**

​例如一个学校图书馆的管理系统，如果是普通学生登录就能看到借书还书相关的功能，不可能让他看到并且去使用添加书籍信息，删除书籍信息等功能。但是如果是一个图书馆管理员的账号登录了，应该就能看到并使用添加书籍信息，删除书籍信息等功能。

不能只依赖前端去判断用户的权限来选择显示哪些菜单哪些按钮。因为如果只是这样，如果有人知道了对应功能的接口地址就可以不通过前端，直接去发送请求来实现相关功能操作。

所以还需要在后台进行用户权限的判断，判断当前用户是否有相应的权限，必须具有所需权限才能进行相应的操作。

## 授权基本流程

在SpringSecurity中，会使用默认的FilterSecurityInterceptor来进行权限校验。它会从SecurityContextHolder获取其中的Authentication，然后获取其中的权限信息。然后判断当前用户是否拥有访问当前接口/资源所需的权限，如果有才能访问

**所以实现授权的方法**

1. 把当前登录用户的权限信息存入Authentication
   * UserDetailsService的loadUserByUsername()方法中，查到用户后，还要把用户拥有的权限也存入UserDetails实现类中返回
   * 认证过滤器中，用户校验成功后，还要把权限信息也放到Authentication里
2. 然后设置各个接口/资源访问所需要的权限

## 授权实现

## 设置访问权限

SpringSecurity提供了基于注解的权限控制方案，用注解去指定访问对应的资源所需的权限。

① SecurityConfig配置类开启相关配置。

```java
@EnableGlobalMethodSecurity(prePostEnabled = true)
```

② ​然后就可以使用对应的注解。

用的比较多的：@PreAuthorize

在访问资源之前进行权限认证

```java
@RestController
public class HelloController {

    @RequestMapping("/hello")
    @PreAuthorize("hasAuthority('test')")
    public String hello(){
        return "hello";
    }
}
```

## 封装权限信息

方便测试：先直接把权限信息写死封装到UserDetails

① 修改UserDetails的实现类LoginUser

* 添加List<String>类型的成员变量用于接收用户拥有的权限
* 添加传入User和权限信息的有参构造
* 重写getAuthorities()。为了只在第一次调用进行集合转换，之后的调用都用第一次的结果，那么将`List<GrantedAuthority>`对象设置为成员变量（**单例模式**）

```java
@Data
@NoArgsConstructor
public class LoginUser implements UserDetails {

    private User user;
        
    //存储权限信息
    private List<String> permissions;
    
    //传入User和权限信息的有参构造
    public LoginUser(User user,List<String> permissions) {
        this.user = user;
        this.permissions = permissions;
    }


    //存储SpringSecurity所需要的权限信息的集合
    @JSONField(serialize = false)//该成员变量不会被序列化到redius中
    private List<GrantedAuthority> authorities;

    @Override
    public  Collection<? extends GrantedAuthority> getAuthorities() {
        if(authorities!=null){
            return authorities;
        }
        //把permissions中字符串类型的权限信息转换成GrantedAuthority对象存入authorities中
        authorities = permissions.stream().
                map(SimpleGrantedAuthority::new)
                .collect(Collectors.toList());
        return authorities;
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUserName();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

② 权限信息封装到UserDetails的实现类LoginUser中

*注意：现在写死权限只是方便测试，实际上要从数据库中查询权限信息*

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(User::getUserName,username);
        User user = userMapper.selectOne(wrapper);
        if(Objects.isNull(user)){
            throw new RuntimeException("用户名或密码错误");
        }
        //TODO 根据用户查询权限信息 添加到LoginUser中
        List<String> list = new ArrayList<>(Arrays.asList("test"));//将权限信息写死方便测试
        return new LoginUser(user,list);
    }
}
```

③ 认证过滤器

第三个参数null改为loginUser.getAuthorities，将权限信息传入

```java
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {

    @Autowired
    private RedisCache redisCache;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
       ....
        //TODO 获取权限信息封装到Authentication中
        UsernamePasswordAuthenticationToken authenticationToken =
                new UsernamePasswordAuthenticationToken(loginUser,null,loginUser.getAuthorities);//选用3个参数的这个，因为它会自动设置把第一个参数设置为已认证状态
        SecurityContextHolder.getContext().setAuthentication(authenticationToken);
        //放行
        filterChain.doFilter(request, response);
    }
```

## 从数据库查询权限信息

之前为了方便测试，权限信息都是写死的，实际上，权限信息应当是从数据库里拿到的

### RBAC权限模型

RBAC权限模型（Role-Based Access Control）即：基于角色的权限控制。这是目前最常被开发者使用也是相对易用、通用权限模型。

![image-20211222110249727](https://image.itbaima.net/images/253/image-20230730152931461.png)

三个主要的表：
* 用户表user
* 权限表menu(权限功能以及权限关键字)，每个权限对应一个功能
* 角色表role(角色名以及角色关键字)，每个角色相当于一组权限的集合

两张关联表（都是多对多的关系）：

角色权限关联表，用户角色关联表

综上，使用RBAC权限模型进行查询，通常是一个五表联查的形式

### 准备工作

```sql

CREATE DATABASE /*!32312 IF NOT EXISTS*/`sg_security` /*!40100 DEFAULT CHARACTER SET utf8mb4 */;

USE `sg_security`;

/*Table structure for table `sys_menu` */

DROP TABLE IF EXISTS `sys_menu`;

CREATE TABLE `sys_menu` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `menu_name` varchar(64) NOT NULL DEFAULT 'NULL' COMMENT '菜单名',
  `path` varchar(200) DEFAULT NULL COMMENT '路由地址',
  `component` varchar(255) DEFAULT NULL COMMENT '组件路径',
  `visible` char(1) DEFAULT '0' COMMENT '菜单状态（0显示 1隐藏）',
  `status` char(1) DEFAULT '0' COMMENT '菜单状态（0正常 1停用）',
  `perms` varchar(100) DEFAULT NULL COMMENT '权限标识',
  `icon` varchar(100) DEFAULT '#' COMMENT '菜单图标',
  `create_by` bigint(20) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `update_by` bigint(20) DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  `del_flag` int(11) DEFAULT '0' COMMENT '是否删除（0未删除 1已删除）',
  `remark` varchar(500) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COMMENT='菜单表';

/*Table structure for table `sys_role` */

DROP TABLE IF EXISTS `sys_role`;

CREATE TABLE `sys_role` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(128) DEFAULT NULL,
  `role_key` varchar(100) DEFAULT NULL COMMENT '角色权限字符串',
  `status` char(1) DEFAULT '0' COMMENT '角色状态（0正常 1停用）',
  `del_flag` int(1) DEFAULT '0' COMMENT 'del_flag',
  `create_by` bigint(200) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `update_by` bigint(200) DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  `remark` varchar(500) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COMMENT='角色表';

/*Table structure for table `sys_role_menu` */

DROP TABLE IF EXISTS `sys_role_menu`;

CREATE TABLE `sys_role_menu` (
  `role_id` bigint(200) NOT NULL AUTO_INCREMENT COMMENT '角色ID',
  `menu_id` bigint(200) NOT NULL DEFAULT '0' COMMENT '菜单id',
  PRIMARY KEY (`role_id`,`menu_id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4;

/*Table structure for table `sys_user` */

DROP TABLE IF EXISTS `sys_user`;

CREATE TABLE `sys_user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `user_name` varchar(64) NOT NULL DEFAULT 'NULL' COMMENT '用户名',
  `nick_name` varchar(64) NOT NULL DEFAULT 'NULL' COMMENT '昵称',
  `password` varchar(64) NOT NULL DEFAULT 'NULL' COMMENT '密码',
  `status` char(1) DEFAULT '0' COMMENT '账号状态（0正常 1停用）',
  `email` varchar(64) DEFAULT NULL COMMENT '邮箱',
  `phonenumber` varchar(32) DEFAULT NULL COMMENT '手机号',
  `sex` char(1) DEFAULT NULL COMMENT '用户性别（0男，1女，2未知）',
  `avatar` varchar(128) DEFAULT NULL COMMENT '头像',
  `user_type` char(1) NOT NULL DEFAULT '1' COMMENT '用户类型（0管理员，1普通用户）',
  `create_by` bigint(20) DEFAULT NULL COMMENT '创建人的用户id',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` bigint(20) DEFAULT NULL COMMENT '更新人',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `del_flag` int(11) DEFAULT '0' COMMENT '删除标志（0代表未删除，1代表已删除）',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COMMENT='用户表';

/*Table structure for table `sys_user_role` */

DROP TABLE IF EXISTS `sys_user_role`;

CREATE TABLE `sys_user_role` (
  `user_id` bigint(200) NOT NULL AUTO_INCREMENT COMMENT '用户id',
  `role_id` bigint(200) NOT NULL DEFAULT '0' COMMENT '角色id',
  PRIMARY KEY (`user_id`,`role_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

```mysql
SELECT 
	DISTINCT m.`perms`
FROM
	sys_user_role ur
	LEFT JOIN `sys_role` r ON ur.`role_id` = r.`id`
	LEFT JOIN `sys_role_menu` rm ON ur.`role_id` = rm.`role_id`
	LEFT JOIN `sys_menu` m ON m.`id` = rm.`menu_id`
WHERE
	user_id = 2
	AND r.`status` = 0
	AND m.`status` = 0
```

```java
/**
 * 菜单表(Menu)实体类
 *
 * @author makejava
 * @since 2021-11-24 15:30:08
 */
@TableName(value="sys_menu")
@Data
@AllArgsConstructor
@NoArgsConstructor
@JsonInclude(JsonInclude.Include.NON_NULL)
public class Menu implements Serializable {
    private static final long serialVersionUID = -54979041104113736L;
    
        @TableId
    private Long id;
    /**
    * 菜单名
    */
    private String menuName;
    /**
    * 路由地址
    */
    private String path;
    /**
    * 组件路径
    */
    private String component;
    /**
    * 菜单状态（0显示 1隐藏）
    */
    private String visible;
    /**
    * 菜单状态（0正常 1停用）
    */
    private String status;
    /**
    * 权限标识
    */
    private String perms;
    /**
    * 菜单图标
    */
    private String icon;
    
    private Long createBy;
    
    private Date createTime;
    
    private Long updateBy;
    
    private Date updateTime;
    /**
    * 是否删除（0未删除 1已删除）
    */
    private Integer delFlag;
    /**
    * 备注
    */
    private String remark;
}
```

### 代码实现

只需要根据用户id去查询到其所对应的权限信息即可。

先定义个mapper，其中提供一个方法可以根据userid查询权限信息

```java
public interface MenuMapper extends BaseMapper<Menu> {
    List<String> selectPermsByUserId(Long id);
}
```

创建对应的mapper映射文件，定义对应的sql语句

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.sangeng.mapper.MenuMapper">


    <select id="selectPermsByUserId" resultType="java.lang.String">
        SELECT
            DISTINCT m.`perms`
        FROM
            sys_user_role ur
            LEFT JOIN `sys_role` r ON ur.`role_id` = r.`id`
            LEFT JOIN `sys_role_menu` rm ON ur.`role_id` = rm.`role_id`
            LEFT JOIN `sys_menu` m ON m.`id` = rm.`menu_id`
        WHERE
            user_id = #{userid}
            AND r.`status` = 0
            AND m.`status` = 0
    </select>
</mapper>
```

​在application.yml中配置mapper映射文件的位置

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/sg_security?characterEncoding=utf-8&serverTimezone=UTC
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
  redis:
    host: localhost
    port: 6379
mybatis-plus:
  mapper-locations: classpath*:/mapper/**/*.xml 
```

然后在UserDetailsServiceImpl中去调用该mapper的方法,查询权限信息封装到LoginUser对象中

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private MenuMapper menuMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(User::getUserName,username);
        User user = userMapper.selectOne(wrapper);
        if(Objects.isNull(user)){
            throw new RuntimeException("用户名或密码错误");
        }
        List<String> permissionKeyList =  menuMapper.selectPermsByUserId(user.getId());
//        //测试写法
//        List<String> list = new ArrayList<>(Arrays.asList("test"));
        return new LoginUser(user,permissionKeyList);
    }
}
```

# 自定义失败处理

希望在认证失败或者是授权失败的情况下也能和我们的接口一样返回相同结构的json，从而让前端能对响应进行统一的处理。

**首先要了解SpringSecurity的异常处理机制**

在SpringSecurity中，如果在认证或者授权的过程中出现了异常会被ExceptionTranslationFilter捕获到。它会去判断是认证失败还是授权失败出现的异常。

**认证过程中出现的异常**会被封装成AuthenticationException然后调用**AuthenticationEntryPoint**对象的方法去进行异常处理。

**授权过程中出现的异常**会被封装成AccessDeniedException然后调用**AccessDeniedHandler**对象的方法去进行异常处理。

**综上，自定义异常处理只需要自定义AuthenticationEntryPoint和AccessDeniedHandler然后配置给SpringSecurity**

①自定义实现类

1. @Component注入
2. 创建携带对应状态码和信息的自定义的响应类
3. 将自定义响应类转换为JSON格式
4. 使用WebUtils工具类设置响应体
   ```java
   response.setStatus(200);
            response.setContentType("application/json");
            response.setCharacterEncoding("utf-8");
            response.getWriter().print(string);
    ```

```java
@Component
public class AccessDeniedHandlerImpl implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        ResponseResult result = new ResponseResult(HttpStatus.FORBIDDEN.value(), "权限不足");
        String json = JSON.toJSONString(result);
        WebUtils.renderString(response,json);

    }
}
```

```java
@Component
public class AuthenticationEntryPointImpl implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        ResponseResult result = new ResponseResult(HttpStatus.UNAUTHORIZED.value(), "认证失败请重新登录");
        String json = JSON.toJSONString(result);
        WebUtils.renderString(response,json);
    }
}
```

②配置给SpringSecurity

先注入对应的处理器

```java
    @Autowired
    private AuthenticationEntryPoint authenticationEntryPoint;

    @Autowired
    private AccessDeniedHandler accessDeniedHandler;
```

然后使用HttpSecurity对象的方法去配置。

```java
        //配置异常处理器
        http.exceptionHandling().authenticationEntryPoint(authenticationEntryPoint).
                accessDeniedHandler(accessDeniedHandler);
```

# 跨域

浏览器出于安全的考虑，使用XMLHttpRequest对象发起HTTP请求时必须遵守同源策略(协议、域名、端口号都完全一致)，否则就是跨域的HTTP请求，默认情况下是被禁止的。

前后端分离项目，前端项目和后端项目一般都不是同源的，所以肯定会存在跨域请求的问题。

所以就要处理一下，让前端能进行跨域请求。

①先对SpringBoot配置，运行跨域请求

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

②开启SpringSecurity的跨域访问

由于我们的资源都会收到SpringSecurity的保护，所以想要跨域访问还要让SpringSecurity运行跨域访问。

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //关闭csrf
                .csrf().disable()
                //不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                // 对于登录接口 允许匿名访问
                .antMatchers("/user/login").anonymous()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();

        //添加过滤器
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);

        //配置异常处理器
        http.exceptionHandling()
                //配置认证失败处理器
                .authenticationEntryPoint(authenticationEntryPoint)
                .accessDeniedHandler(accessDeniedHandler);

        //允许跨域
        http.cors();
    }
```

# 补充 

## 其它权限校验方法

前面都是使用@PreAuthorize注解，然后在其中使用的是hasAuthority方法进行校验

SpringSecurity还提供了其它方法例如：hasAnyAuthority，hasRole，hasAnyRole等

**先理解hasAuthority的原理**

hasAnyAuthority方法可以传入多个权限，用户有其中任意一个权限都可以访问对应资源。

hasAuthority方法实际是执行到了SecurityExpressionRoot的hasAuthority，它内部其实是调用authentication的getAuthorities方法获取用户的权限列表。然后判断存入的方法参数数据是否在权限列表中。

```java
    @PreAuthorize("hasAnyAuthority('admin','test','system:dept:list')")
    public String hello(){
        return "hello";
    }
```

**hasRole**

hasRole要求有对应的角色才可以访问。

它内部会把传入的参数拼接上 **ROLE_** 后再去比较。所以这种情况下要用户对应的权限也要有 **ROLE_** 这个前缀才可以。

```java
    @PreAuthorize("hasRole('system:dept:list')")
    public String hello(){
        return "hello";
    }
```

**hasAnyRole**

hasAnyRole有任意的角色就可以访问。

它内部也会把传入的参数拼接上 **ROLE_** 后再去比较。所以这种情况下要用户对应的权限也要有 **ROLE_** 这个前缀才可以。

```java
    @PreAuthorize("hasAnyRole('admin','system:dept:list')")
    public String hello(){
        return "hello";
    }
```

## 自定义权限校验方法

定义自己的权限校验方法，在@PreAuthorize注解中使用自定义的方法。

```java
@Component("ex")//将注入的bean名称设置为“ex”
public class SGExpressionRoot {

    public boolean hasAuthority(String authority){
        //获取当前用户的权限
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        LoginUser loginUser = (LoginUser) authentication.getPrincipal();
        List<String> permissions = loginUser.getPermissions();
        //判断用户权限集合中是否存在authority
        return permissions.contains(authority);
    }
}
```

在SPEL表达式中使用`@ex.hasAuthority`相当于获取容器中bean的名字为ex的对象,然后再调用这个对象的hasAuthority方法（这样才不会去执行默认的hasAuthority方法）

```java
    @RequestMapping("/hello")
    @PreAuthorize("@ex.hasAuthority('system:dept:list')")
    public String hello(){
        return "hello";
    }
```

## 基于配置的权限控制

除了用注解设置访问权限外，还可以在配置类中，使用配置的方式对资源进行权限控制。

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //关闭csrf
                .csrf().disable()
                //不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                // 对于登录接口 允许匿名访问
                .antMatchers("/user/login").anonymous()
                .antMatchers("/testCors").hasAuthority("system:dept:list222")
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();

        //添加过滤器
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);

        //配置异常处理器
        http.exceptionHandling()
                //配置认证失败处理器
                .authenticationEntryPoint(authenticationEntryPoint)
                .accessDeniedHandler(accessDeniedHandler);

        //允许跨域
        http.cors();
    }
```

## CSRF

CSRF是指跨站请求伪造（Cross-site request forgery），是web常见的攻击之一。

https://blog.csdn.net/freeking101/article/details/86537087

SpringSecurity防止CSRF攻击的方式就是通过csrf_token。后端会生成一个csrf_token，前端发起请求的时候需要携带这个csrf_token。后端会有过滤器进行校验，如果没有携带或者是伪造的就不允许访问。

CSRF攻击依靠的是cookie中所携带的认证信息。但是在前后端分离的项目中，认证信息其实是token，而token并不是存储中cookie中，并且需要前端代码去把token设置到请求头中才可以，所以CSRF攻击也就不用担心了。
