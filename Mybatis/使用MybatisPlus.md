# MybatisPlus

MybatisPlus是一款Mybatis增强工具，用于简化开发，提高效率。它在 MyBatis的基础上只做增强不做改变。

官网： https://mp.baomidou.com/ 

## 快速入门

### 准备工作

①准备数据

~~~~mysql
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `user_name` varchar(20) NOT NULL COMMENT '用户名',
  `password` varchar(20) NOT NULL COMMENT '密码',
  `name` varchar(30) DEFAULT NULL COMMENT '姓名',
  `age` int(11) DEFAULT NULL COMMENT '年龄',
  `address` varchar(100) DEFAULT NULL COMMENT '地址',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8;

insert  into `user`(`id`,`user_name`,`password`,`name`,`age`,`address`) values (1,'ruiwen','123','瑞文',12,'山东'),(2,'gailun','1332','盖伦',13,'平顶山'),(3,'timu','123','提姆',22,'蘑菇石'),(4,'daji','1222','妲己',221,'狐山');
~~~~

②创建SpringBoot工程

添加依赖

~~~~xml
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
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
~~~~

创建启动类

~~~~java
@SpringBootApplication
public class SGApplication {

    public static void main(String[] args) {
        SpringApplication.run(SGApplication.class);
    }
}

~~~~

③准备实体类

~~~~java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private Long id;
    private String userName;
    private String password;
    private String name;
    private Integer age;
    private String address;
}
~~~~

### 使用MybatisPlus

①添加依赖

~~~~xml
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
~~~~


②配置数据库信息

~~~~yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mp_db?characterEncoding=utf-8&serverTimezone=UTC
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
~~~~

③创建Mapper接口

**创建Mapper接口继承BaseMapper接口**

BaseMapper接口中已经提供了很多常用方法，只需要直接从容器中获取Mapper就可以进行操作了，不需要自己去编写Sql语句。

~~~~java
public interface UserMapper extends BaseMapper<User> {
}
~~~~

④配置Mapper扫描

在启动类上配置@MapperScan

~~~~java
@SpringBootApplication
@MapperScan("com.sangeng.mapper")
public class SGApplication {
    public static void main(String[] args) {
        SpringApplication.run(SGApplication.class,args);
    }
}

~~~~

⑤获取Mapper进行测试

~~~~java
@SpringBootTest
public class MPTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testQueryList(){
        System.out.println(userMapper.selectList(null));
    }

}
~~~~

## 常用设置

### 设置表与实体类的映射

默认情况下MP操作的表名就是实体类的类名，但是如果表名和类名不一致就需要自己设置映射规则。

#### 单独设置

在实体类的类名上加上@TableName注解进行标识

例如：

表名是tb_user，而实体类名是User则可以使用以下写法。

~~~~java
@TableName("tb_user")
public class User {
	//....
}

~~~~

#### 全局设置表名前缀

一般一个项目表名的前缀都是统一风格的，这个时候如果一个个设置就太麻烦了。

例如：

如果一个项目中所有的表名相比于类名都是多了个前缀： `tb_` 。这可以使用如下方式配置

~~~~yml
mybatis-plus:
  global-config:
    db-config:
      #表名前缀
      table-prefix: tb_
~~~~

### 设置主键生成策略

测试代码

~~~~java
    @Test
    public void testInsert(){
        User user = new User();
        user.setUserName("水水水水");
        user.setPassword("755");
        int r = userMapper.insert(user);
        System.out.println(r);
    }
~~~~

#### 单独设置

默认情况下使用MP插入数据时，在我们没有设置主键生成策略的情况下，默认的策略是基于雪花算法的自增id。

如果需要使用别的策略可以在定义实体类时，在代表主键的字段上加上`@TableId`注解，使用其`type`属性指定主键生成策略。

例如：

要设置主键自动增长则可以设置如下

~~~~java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
	//.....
}

~~~~

全部主键策略定义在了枚举类`IdType`中，`IdType`有如下的取值

- `AUTO`

  数据库ID自增，**依赖于数据库**。该类型要确保数据库设置了ID自增，否则无效

  

- `NONE`

  未设置主键类型。若在代码中没有手动设置主键，则会根据**主键的全局策略**自动生成（默认的主键全局策略是基于雪花算法的自增ID）

  

- `INPUT`

  需要手动设置主键，若不设置。插入操作生成SQL语句时，主键这一列的值会是`null`。

  

- `ASSIGN_ID`

  当没有手动设置主键，即实体类中的主键属性为空时，才会自动填充，使用雪花算法

  

- `ASSIGN_UUID`

  当实体类的主键属性为空时，才会自动填充，使用UUID

#### 全局设置

配置这个项目所有id生成策略

~~~~yaml
mybatis-plus:
  global-config:
    db-config:
      # id生成策略 auto为数据库自增
      id-type: auto
~~~~

### 设置字段映射关系

默认情况下MP会根据实体类的属性名去映射表的列名。

如果数据库的列表和实体类的属性名不一致，可以使用`@TableField`注解的`value`属性去设置映射关系。

​
例如：

表中一个列名叫address而实体类中的属性名为addressStr

~~~~java
    @TableField("address")
    private String addressStr;
~~~~

### 设置字段和列名的驼峰映射

默认情况下MP会开启字段名列名的驼峰映射，即从经典数据库列名A_COLUMN（下划线命名）到经典Java属性名aColumn（驼峰命名）的类似映射

如果需要关闭，可以使用如下配置进行关闭。

~~~~yaml
mybatis-plus:
  configuration:
	#是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN（下划线命名） 到经典 Java 属性名 aColumn（驼峰命名） 的类似映射
    map-underscore-to-camel-case: false
~~~~

### 日志

如果需要打印MP操作对应的SQL语句等，可以配置日志输出。

配置方式如下：

~~~~yml
mybatis-plus:
  configuration:
    # 日志
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
~~~~

## 基本使用

### 插入数据

使用insert方法来实现数据的插入。

示例：

~~~~java
    @Test
    public void testInsert(){
        User user = new User();
        user.setUserName("efaf");
        user.setPassword("31432");
        int r = userMapper.insert(user);
        System.out.println(r);
    }
~~~~

### 删除操作

使用**deleteXXX**方法来实现数据的删除。

示例：

~~~~java
    @Test
    public void testDelete(){
        List<Integer> ids = new ArrayList<>();
        ids.add(5);
        ids.add(6);
        ids.add(7);
        int i = userMapper.deleteBatchIds(ids);
        System.out.println(i);
    }
    @Test
    public void testDeleteById(){
        int i = userMapper.deleteById(8);
        System.out.println(i);
    }
    @Test
    public void testDeleteByMap(){
        Map<String, Object> map = new HashMap<>();
        map.put("name","提姆");
        map.put("age",22);
        int i = userMapper.deleteByMap(map);
        System.out.println(i);
    }
~~~~

### 更新操作

使用**updateXXX**方法来实现数据的删除。

示例：

~~~~java
    @Test
    public void testUpdate(){
        //把id为2的用户的年龄改为14
        User user = new User();
        user.setId(2L);//id是Long类型
        user.setAge(14);
        int i = userMapper.updateById(user);
        System.out.println(i);
    }
~~~~

## 条件构造器Wrapper

实际操作数据库的时候会涉及到很多的条件。

MP提供了一个功能强大的条件构造器 `Wrapper`，使用它可以非常方便地构造条件。

其继承体系如下：

![image-20210823105447044](img\wrapper继承体系图.png)

在其子类`AbstractWrapper`中提供了很多用于构造Where条件的方法。

`AbstractWrapper`的子类`QueryWrapper`额外提供了用于针对Select语法的`select`方法。可以用来设置查询哪些列。

`AbstractWrapper`的子类`UpdateWrapper`额外提供了用于针对SET语法的`set`方法。可以用来设置对哪些列进行更新。

完整的AbstractWrapper方法可以参照：https://baomidou.com/guide/wrapper.html#abstractwrapper

介绍是用来干什么的。它的实现类有哪些

QueryWrapper,UpdateWrapper，【LambdaQueryWrapper】

### AbstractWrapper

> eq：equals，等于
> gt：greater than ，大于 >
> ge：greater than or equals，大于等于≥
> lt：less than，小于<
> le：less than or equals，小于等于≤
> between：相当于SQL中的BETWEEN
> like：模糊匹配。like("name","黄")，相当于SQL的name like '%黄%'
> likeRight：模糊匹配右半边。likeRight("name","黄")，相当于SQL的name like '黄%'
> likeLeft：模糊匹配左半边。likeLeft("name","黄")，相当于SQL的name like '%黄'
> notLike：notLike("name","黄")，相当于SQL的name not like '%黄%'
> isNull
> isNotNull
> and：SQL连接符AND
> or：SQL连接符OR
>
> in: in(“age",{1,2,3})相当于 age in(1,2,3)
>
> groupBy: groupBy("id","name")相当于 group by id,name
>
> orderByAsc :orderByAsc("id","name")相当于 order by id ASC,name ASC
>
> orderByDesc :orderByDesc ("id","name")相当于 order by id DESC,name DESC

#### 示例一

SQL语句如下： 

~~~~mysql
SELECT 
	id,user_name,PASSWORD,NAME,age,address 
FROM 
	USER 
WHERE 
	age > 18 AND address = '狐山'
~~~~

如果用Wrapper写法如下：

~~~~java
    @Test
    public void testWrapper01(){
        QueryWrapper wrapper = new QueryWrapper();
        wrapper.gt("age",18);
        wrapper.eq("address","狐山");
        List<User> users = userMapper.selectList(wrapper);
        System.out.println(users);
    }
~~~~

#### 示例二

SQL语句如下：

~~~~mysql
SELECT 
	id,user_name,PASSWORD,NAME,age,address 
FROM 
	USER 
WHERE 
	id IN(1,2,3) AND 
	age BETWEEN 12 AND 29 AND 
	address LIKE '%山%'
~~~~

如果用Wrapper写法如下：

~~~~java
    @Test
    public void testWrapper02(){
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.in("id",1,2,3);
        wrapper.between("age",12,29);
        wrapper.like("address","山");
        List<User> users = userMapper.selectList(wrapper);
        System.out.println(users);
    }
~~~~

#### 示例三

SQL语句如下：

~~~~mysql
SELECT 
	id,user_name,PASSWORD,NAME,age,address 
FROM 
	USER 
WHERE 
	id IN(1,2,3) AND 
	age > 10 
ORDER BY 
	age DESC
~~~~

如果用Wrapper写法如下：

~~~~java
    @Test
    public void testWrapper03(){
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.in("id",1,2,3);
        queryWrapper.gt("age",10);
        queryWrapper.orderByDesc("age");
        List<User> users = userMapper.selectList(queryWrapper);
        System.out.println(users);
    }
~~~~

### QueryWrapper

QueryWrapper的select可以**设置要查询的列**

#### 示例一 

> select(String... sqlSelect) 方法的测试为要查询的列名

SQL语句如下：

~~~~mysql
SELECT 
	id,user_name
FROM 
	USER 
~~~~

MP写法如下：

~~~~java
    @Test
    public void testSelect01(){
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.select("id","user_name");
        List<User> users = userMapper.selectList(queryWrapper);
        System.out.println(users);
    }
~~~~


#### 示例二

> select(Class<T> entityClass, Predicate<TableFieldInfo> predicate)

方法的第一个参数为实体类的字节码对象

第二个参数为Predicate类型，可以使用lambda的写法，过滤要查询的字段 (主键除外)

SQL语句如下：

~~~~mysql
SELECT 
	id,user_name
FROM 
	USER 
~~~~

MP写法如下：

~~~~java
    @Test
    public void testSelect02(){
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.select(User.class, new Predicate<TableFieldInfo>() {
            @Override
            public boolean test(TableFieldInfo tableFieldInfo) {
                return "user_name".equals(tableFieldInfo.getColumn());
            }
        });
        List<User> users = userMapper.selectList(queryWrapper);
        System.out.println(users);
    }
~~~~

#### 示例三

> select(Predicate<TableFieldInfo> predicate)

方法第一个参数为Predicate类型，可以使用lambda的写法，过滤要查询的字段 (主键除外) 。


SQL语句如下：

~~~~mysql
SELECT 
	id,user_name,PASSWORD,NAME,age 
FROM 
	USER
~~~~

不想查询address这列，其他列都查询了（这在表有很多列的时候比较方便）



MP写法如下：

~~~~java
    @Test
    public void testSelect03(){
        QueryWrapper<User> queryWrapper = new QueryWrapper<>(new User());
        queryWrapper.select(new Predicate<TableFieldInfo>() {
            @Override
            public boolean test(TableFieldInfo tableFieldInfo) {
                return !"address".equals(tableFieldInfo.getColumn());
            }
        });
        List<User> users = userMapper.selectList(queryWrapper);
        System.out.println(users);
    }
~~~~

### UpdateWrapper

之前在使用update方法时需要创建一个实体类对象传入，用来指定要更新的列及对应的值。但是如果需要更新的列比较少时，创建这么一个对象显的有点麻烦和复杂。

使用UpdateWrapper的set方法可以设置要更新的列及其值。同时这种方式也可以使用Wrapper去指定更复杂的更新条件。

#### 示例

SQL语句如下：

~~~~mysql
UPDATE 
	USER
SET 
	age = 99
where 
	id > 1
~~~~

把id大于1的用户的年龄修改为99，则可以使用如下写法：

~~~~java
    @Test
    public void testUpdateWrapper(){
        UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
        updateWrapper.gt("id",1);
        updateWrapper.set("age",99);
        userMapper.update(null,updateWrapper);
    }
~~~~

### Lambda条件构造器

前面在使用条件构造器时列名都是用字符串的形式去指定。这种方式**无法在编译期确定列名的合法性**

所以MP提供了一个Lambda条件构造器可以直接以“实体类的方法引用”的形式来指定列名。这样就可以弥补上述缺陷。

**字符串指定列名**-->**通过实体类的方法引用指定列名**

#### 示例		

要执行的查询对应的SQL如下

~~~~mysql
SELECT 
	id,user_name,PASSWORD,NAME,age,address 
FROM 
	USER 
WHERE 
	age > 18 AND address = '狐山'
~~~~

如果使用之前的条件构造器写法如下

~~~~java
    @Test
    public void testLambdaWrapper(){
        QueryWrapper<User> queryWrapper = new QueryWrapper();
        queryWrapper.gt("age",18);
        queryWrapper.eq("address","狐山");
        List<User> users = userMapper.selectList(queryWrapper);
    }
~~~~

如果使用Lambda条件构造器写法如下

~~~~java
    @Test
    public void testLambdaWrapper2(){
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.gt(User::getAge,18);
        queryWrapper.eq(User::getAddress,"狐山");
        List<User> users = userMapper.selectList(queryWrapper);
    }
~~~~

## 自定义SQL

虽然MP提供了很多常用的方法，并且也提供了条件构造器。但是如果真的遇到了复制的SQL时，还是需要自己去定义方法，自己去写对应的SQL，这样SQL也更有利于后期维护。

因为MP是对mybatis做了增强，所以还是支持之前Mybatis的方式去自定义方法。

同时也支持在使用Mybatis的自定义方法时使用MP的条件构造器帮助我们进行条件构造。

### 准备工作

①准备数据

~~~~mysql
CREATE TABLE `orders` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `price` int(11) DEFAULT NULL COMMENT '价格',
  `remark` varchar(100) DEFAULT NULL COMMENT '备注',
  `user_id` int(11) DEFAULT NULL COMMENT '用户id',
  `update_time` timestamp NULL DEFAULT NULL COMMENT '更新时间',
  `create_time` timestamp NULL DEFAULT NULL COMMENT '创建时间',
  `version` int(11) DEFAULT '1' COMMENT '版本',
  `del_flag` int(1) DEFAULT '0' COMMENT '逻辑删除标识,0-未删除,1-已删除',
  `create_by` varchar(100) DEFAULT NULL COMMENT '创建人',
  `update_by` varchar(100) DEFAULT NULL COMMENT '更新人',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8;

/*Data for the table `orders` */

insert  into `orders`(`id`,`price`,`remark`,`user_id`,`update_time`,`create_time`,`version`,`del_flag`,`create_by`,`update_by`) values (1,2000,'无',2,'2021-08-24 21:02:43','2021-08-24 21:02:46',1,0,NULL,NULL),(2,3000,'无',3,'2021-08-24 21:03:32','2021-08-24 21:03:35',1,0,NULL,NULL),(3,4000,'无',2,'2021-08-24 21:03:39','2021-08-24 21:03:41',1,0,NULL,NULL);

~~~~

②创建实体类

~~~~java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Orders  {

    private Long id;

    /**
     * 价格
     */
    private Integer price;

    /**
     * 备注
     */
    private String remark;

    /**
     * 用户id
     */
    private Integer userId;

    /**
     * 更新时间
     */
    private LocalDateTime updateTime;

    /**
     * 创建时间
     */
    private LocalDateTime createTime;

    /**
     * 版本
     */
    private Integer version;

    /**
     * 逻辑删除标识,0-未删除,1-已删除
     */
    private Integer delFlag;


}
~~~~

### Mybatis方式

#### ①定义方法

在Mapper接口中定义方法

~~~~java
public interface UserMapper extends BaseMapper<User> {

    User findMyUser(Long id);
}

~~~~

#### ②创建xml

先配置xml文件的存放目录

~~~~yml
mybatis-plus:
  mapper-locations: classpath*:/mapper/**/*.xml
~~~~

创建对应的xml映射文件

#### ③在xml映射文件中编写SQL

创建对应的标签，编写对应的SQL语句

~~~~java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.sangeng.mapper.UserMapper">

    <select id="findMyUser" resultType="com.sangeng.domian.User">
       select * from user where id = #{id} 
    </select>
</mapper>
~~~~

### 结合条件构造器

**Wrapper对象的CustomSqlSegment属性保存着最终构建完的sql语句片段，因此只要获取到它，然后把它拼接到sql语句中即可**

①方法定义中添加Warpper类型的参数

添加`Warpper`类型的参数，并且要注意给其指定参数名(可以随便写)

~~~~java
public interface UserMapper extends BaseMapper<User> {

    //Constants.WRAPPER是常量"ew"
    User findMyUserByWrapper(@Param(Constants.WRAPPER) Wrapper<User> wrapper);
}
~~~~


②在SQL语句中获取Warpper拼接的SQL片段进行拼接。

**注意：不能使用#{}应该用${}**

~~~~xml
    <select id="findMyUserByWrapper" resultType="com.sangeng.domian.User">
        select * from user ${ew.customSqlSegment}
    </select>
~~~~

③ 测试

```java
 @Test
    public void testFindMyUserByWrapper(){
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.gt(User::getAge,18);
        queryWrapper.eq(User::getAddress,"狐山");
        User myUser = userMapper.findMyUserByWrapper(queryWrapper);
        System.out.println(myUser);
    }
```

## 分页查询

### 基本分页查询

①配置分页查询拦截器

~~~~java
@Configuration
public class PageConfig {

    /**
     * 3.4.0之前的版本
     * @return
     */
   /* @Bean
    public PaginationInterceptor paginationInterceptor(){
        return  new PaginationInterceptor();
    }*/

    /**
     * 3.4.0之后版本
     * @return
     */
   @Bean
   public MybatisPlusInterceptor mybatisPlusInterceptor(){
       MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
       mybatisPlusInterceptor.addInnerInterceptor(new PaginationInnerInterceptor());
       return mybatisPlusInterceptor;
   }
}
~~~~

②进行分页查询

~~~~java
    @Test
    public void testPage(){
        IPage<User> page = new Page<>();
        //设置每页条数
        page.setSize(2);
        //设置查询第几页
        page.setCurrent(1);
        userMapper.selectPage(page, null);
        System.out.println(page.getRecords());//获取当前页的数据
        System.out.println(page.getTotal());//获取总记录数
        System.out.println(page.getCurrent());//当前页码
    }
~~~~

### 多表分页查询

如果需要在多表查询时进行分页查询的话，就可以在mapper接口中**自定义方法**，然后让方法接收Page对象。

#### 示例

需求：

查询Orders表，并且要求查询的时候除了要获取到Orders表中的字段，还要获取到每个订单的下单用户的用户名。

##### 准备工作

~~~mysql
SELECT 
	o.*,u.`user_name`
FROM 
	USER u,orders o
WHERE 
	o.`user_id` = u.`id`
~~~

实体类修改

增加一个userName属性

~~~~java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Orders  {
	//省略无关代码
    private String userName;
}
~~~~

##### 实现

①定义接口，定义方法

方法返回值是IPage类型，方法第一个参数定义成Page类型

~~~~java
public interface OrdersMapper extends BaseMapper<Orders> {

    IPage<Orders> findAllOrders(Page<Orders> page);
}
~~~~


*在xml中不需要关心分页操作*，MP会帮我们完成

~~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.sangeng.mapper.OrdersMapper">
    <select id="findAllOrders" resultType="com.sangeng.domian.Orders">
        SELECT
            o.*,u.`user_name`
        FROM
            USER u,orders o
        WHERE
            o.`user_id` = u.`id`
    </select>
</mapper>
~~~~

然后调用方法测试即可

~~~~java
    @Autowired
    private OrdersMapper ordersMapper;
    @Test
    public void testOrdersPage(){
        Page<Orders> page = new Page<>();
        //设置每页大小
        page.setSize(2);
        //设置当前页码
        page.setCurrent(2);
        ordersMapper.findAllOrders(page);
        System.out.println(page.getRecords());
        System.out.println(page.getTotal());
    }
~~~~

## Service 层接口

MP提供了Service层的实现。

1. 编写一个service接口，继承`IService`
2. 创建对应的实现类，并继承`ServiceImpl`

相比于Mapper接口，Service层主要是支持了更多批量操作的方法。

### 基本使用

#### 改造前

定义接口

~~~~java
public interface UserService {
    List<User> list();
}

~~~~

定义实现类，注入mapper接口

~~~~java
@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserMapper userMapper;

    @Override
    public List<User> list() {
        return userMapper.selectList(null);
    }
}

~~~~

#### 改造后

接口

继承`IService`

~~~~java
public interface UserService extends IService<User> {

}
~~~~

实现类

实现接口，并继承`ServiceImpl`

~~~~java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper,User> implements UserService {
	
}

~~~~

测试

~~~~java
    @Autowired
    private UserService userService;


    @Test
    public void testSeervice(){
        List<User> list = userService.list();
        System.out.println(list);
    }
~~~~



### 自定义方法

mp提供的service方法不满足需求，需要自定义方法

~~~~java
public interface UserService extends IService<User> {


    User test();
}

~~~~

要用到别的mapper，就注入

用到自己的ServiceImpl里面的mapper可以用getBaseMapper()方法

~~~~java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper,User> implements UserService {

    @Autowired
    private OrdersMapper ordersMapper;

    @Override
    public User test() {
        UserMapper userMapper = getBaseMapper();//相当于注入UserMapper对象
        List<Orders> orders = ordersMapper.selectList(null);
        User user = userMapper.selectById(3);
        //查询用户对于的订单
        QueryWrapper<Orders> wrapper = new QueryWrapper<>();
        wrapper.eq("user_id",3);
        List<Orders> ordersList = ordersMapper.selectList(wrapper);
        return user;
    }
}
~~~~

## 代码生成器

MP提供了一个代码生成器，可以一键生成实体类，Mapper接口，Service，Controller等全套代码

### ①添加依赖

~~~~xml
        <!--mybatisplus代码生成器-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
            <version>3.4.1</version>
        </dependency>
        <!--模板引擎-->
        <dependency>
            <groupId>org.freemarker</groupId>
            <artifactId>freemarker</artifactId>
        </dependency>
~~~~

### ②生成

配置代码生成器（工具项目不同对下面代码进行对应调整即可）

~~~~java

public class GeneratorTest {
	@Test
	public void generate() {
		AutoGenerator generator = new AutoGenerator();

		// 全局配置
		GlobalConfig config = new GlobalConfig();
		String projectPath = System.getProperty("user.dir");
		// 设置输出到的目录
		config.setOutputDir(projectPath + "/src/main/java");
		config.setAuthor("sangeng");
		// 生成结束后是否打开文件夹
		config.setOpen(false);

		// 全局配置添加到 generator 上
		generator.setGlobalConfig(config);

		// 数据源配置
		DataSourceConfig dataSourceConfig = new DataSourceConfig();
		dataSourceConfig.setUrl("jdbc:mysql://localhost:3306/mp_db?characterEncoding=utf-8&serverTimezone=UTC");
		dataSourceConfig.setDriverName("com.mysql.cj.jdbc.Driver");
		dataSourceConfig.setUsername("root");
		dataSourceConfig.setPassword("root");

		// 数据源配置添加到 generator
		generator.setDataSource(dataSourceConfig);

		// 包配置, 生成的代码放在哪个包下
		PackageConfig packageConfig = new PackageConfig();
		packageConfig.setParent("com.sangeng.mp.generator");

		// 包配置添加到 generator
		generator.setPackageInfo(packageConfig);

		// 策略配置
		StrategyConfig strategyConfig = new StrategyConfig();
		// 下划线驼峰命名转换
		strategyConfig.setNaming(NamingStrategy.underline_to_camel);
		strategyConfig.setColumnNaming(NamingStrategy.underline_to_camel);
		// 开启lombok
		strategyConfig.setEntityLombokModel(true);
		// 开启RestController
		strategyConfig.setRestControllerStyle(true);
		generator.setStrategy(strategyConfig);
		generator.setTemplateEngine(new FreemarkerTemplateEngine());

        // 开始生成
		generator.execute();
	}
}
~~~~

## 自动填充

在实际项目中的表和orders表一样，有**更新时间**，**创建时间**，**创建人，更新人**等字段。

使用 `@TableField` 的 `fill` 属性来设置字段的自动填充，能更方便地更新相关字段。

**示例**

>哪些字段填充？什么操作的时候填充？填充什么？

①在对应字段上增加注解

* TableField注解的fill属性标注哪些字段需要在自动填充
* fill属性值代表在进行什么操作时需要预留字段。

~~~~java
    /**
     * 更新时间
     */
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;

    /**
     * 创建时间
     */
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
~~~~

②自定义填充处理器：实现MetaObjectHandler接口 

~~~~java
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        this.setFieldValByName("createTime", LocalDateTime.now(), metaObject);
        this.setFieldValByName("updateTime", LocalDateTime.now(), metaObject);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.setFieldValByName("updateTime", LocalDateTime.now(), metaObject);
    }
}

~~~~

## 逻辑删除

**实际项目中，数据不会进行真正的删除，而是用一个标记代表它是否被删除（逻辑删除）**

### 逻辑删除的好处

1. 利于数据备份和恢复
在物理删除的情况下，一旦数据被删除，就无法恢复。而逻辑删除则只是将数据标记为删除状态，实际上并没有真正地从数据库中删除数据。如果将来需要恢复数据，只需要将其状态修改回来就可以了。

2. 数据的完整性
在使用物理删除操作时，如果有其他表与被删除的数据存在关联，那么这些关联数据可能会出现异常现象，导致数据库数据不完整。而逻辑删除则**配合相关的级联删除**操作可以在保证数据的完整性的同时删除数据。

3. 防止误删
逻辑删除的操作不会真正删除数据，就算用户误删了数据，也可以通过修改状态来将数据重新恢复。

4. 安全性
在数据库中，一些敏感数据被误删后可能会导致严重的后果。使用逻辑删除操作可以保证数据的安全性

### 关联表问题

实体表某数据进行逻辑删除后，其关联表怎么办？

**方案一**

关联表也添加逻辑删除字段，进行逻辑删除

并且采用级联删除的机制：当主表中的记录被删除时，相关联的从表中的记录也会被自动删除，这样就能够保证数据始终处于一致的状态。

**方案二**

如果在实际的业务处理过程中，这些数据并不会被频繁查询或使用到，那么可以选择保留关联表数据，即不逻辑删除，也不物理删除。

以上三种方案都有各自优缺点，需要根据实际去权衡。

比如：

* 如果只进行逻辑删除，关联表中的外键约束依然存在，但是关联表的数据并不会被删除。这可能导致在查询关联表中的数据时会包含已逻辑删除的数据，或者在维护关联表数据时会出现一些逻辑上的混乱。


### MP中使用逻辑删除

MP支持逻辑删除的处理：

配置逻辑删除的实体字段名，代表已删除的字段值，代表未删除的字段值

*注意：如果3.3.0版本之前还需要在对应的字段上加上`@TableLogic`注解*

~~~~yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: delFlag  # 全局逻辑删除的实体字段名(since 3.3.0,配置后可以忽略不配置步骤2)
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
~~~~

## 乐观锁

并发操作时,需要保证对数据的操作不发生冲突。

乐观锁就是其中一种方式：先加上不存在并发冲突问题，在进行实际数据操作的时候再检查是否冲突。

在使用乐观锁时一般在表中增加一个version列，用来记录对某条记录操作的版本。

每次对某条记录进行操作后，对应的版本也需要+1。

然后在每次要进行更新操作时，先查询对应数据的version值。在执行更新时，set version = 老版本+1 where version = 老版本 

如果在查询老版本号到更新操作的中间时刻有其他人更新了这条数据，这样这次更新语句就会更新失败。

这里在更新时对version的操作如果由我们自己做就会显的有点麻烦。所以MP提供了乐观锁插件。

### 使用

①配置对应插件

~~~~java
@Configuration
public class MybatisPlusConfig {
    /**
     * 旧版
     */
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor() {
        return new OptimisticLockerInterceptor();
    }
    
    /**
     * 新版
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        mybatisPlusInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return mybatisPlusInterceptor;
    }
}
~~~~

②在实体类的字段上加上`@Version`注解

~~~~java
@Version
private Integer version;
~~~~

③更新

**注意：在更新前一定要先查询到version设置到实体类上再进行更新才能生效**

~~~~java
    @Test
    public void testVersion(){
        //查询id为3的数据
        QueryWrapper<Orders> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("id",3);
        Orders orders = ordersMapper.selectOne(queryWrapper);


        //对id为3的数据进行更新  把price修改为88
        orders.setPrice(88);
        ordersMapper.updateById(orders);
    }
~~~~

这种情况下可以看到执行的sql已经发生了变化。

~~~~
==>  Preparing: UPDATE orders SET price=?, update_time=?, version=? WHERE id=? AND version=? AND del_flag=0
==> Parameters: 8888(Integer), null, 2(Integer), 2(Long), 1(Integer)
~~~~

### 多插件配置问题

在使用3.4.0版本以后的MP时，如果需要用到多个插件的话：

在配置的时候只需要注入一个MybatisPlusInterceptor对象，把插件对象添加到MybatisPlusInterceptor对象中

​	例如：

~~~~java
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return interceptor;
    }
~~~~



