RESTful只是一种接口设计规则，建议遵循，也可以根据具体场景灵活调整

常见的有RESTful框架有 SpringMVC , jersey , play,...

相关的APT测试工具：Postman, Insomnia，ApiFox

# 提出背景

web2.0阶段出现后，客户端将不仅限于PC浏览器，可能是移动APP，也可能是小程序，这就要求服务端能提供一套统一的API接口，不同类型的客户端基于相同的协议/规则就能来调用该API接口并获取预期的数据。

此时核心：如何设计一套科学的API接口？

不同开发者对API接口的设计习惯不同，对于同一功能，可能设计出的api各不相同
```
新增员工:
http://localhost/employee/save
http://localhost/employee/add
http://localhost/employee/new
http://localhost/employee/xinzeng
http://localhost/employee/append
http://localhost/employee?cmd=add
```
而且发送的请求方式以及响应结果也比较可能随意

为了规范化，提出了**RESTful风格**的接口

>REST是一种设计API接口规则(风格)，因其具有简单、易读、易用的特点，在web项目中非常受欢迎。在设计接口时，如果满足rest约束条件和原则的应用程序或设计就称其为RESTful 应用。

# 约束规则

传统的web接口设计
以员工列表为例子。

```java
@Controller
public class EmployeeController {
    @RequestMapping("/employee/list")
    public String list(Model model){
        model.addAttribute("list", employeeService.list())
        return "employee/list";
    }
}
```

设计传统web接口时考虑：

* 请求路径：一般采用见名知意的方式，比如：/employee/list
* 请求方式：不在意，@RequestMapping注解可以接受任意请求方式，包括:GET POST
* 请求参数：根据接口的功能而定，由需求决定
* 请求响应：由需求决定，可以是Json格式也可以页面模板

***

**RESTful风格的接口设计与传统web接口不同**
```java
@Controller
public class EmployeeController {
    @RequestMapping(value = "/employees", method = RequestMethod.GET)
	@ResponseBody
    public List<Employee> list(){
        return employeeService.list();
    }
}
```

## 请求路径

>请求路径一般就是操作实体对象复数

不再是见名知意的方式，而是**由操作的资源决定**，一般采用资源名称复数形式
比如接口操作对象(资源)是员工，路径可设计为：/employees

>一切皆为资源
>RESTful眼中，互联网上，一切皆为资源(图片，网页，请求路径,...)，并且，每个资源都有一个唯一的资源定位符（URI）。

回到示例代码，http://localhost:8080/employee?id=1  表示查询数据库中id=1的员工信息，这个员工信息就是restful中描述的资源。

一般来说，资源不会只有一个，就像员工不仅仅只有id=1的数据，大多都是以复数，所以RESTful 约定：针对接口要操作的资源**统一使用复数**


比如
```
http://www.xxx.cn/depts  部门资源
https://api.example.com/v1/zoos：动物园资源
https://api.example.com/v1/animals：动物资源
https://api.example.com/v1/employees：饲养员资源
```

## 请求方式

传统接口设计方式，使用见名知意设计路径，可以从路径上看出接口对资源操作

而RESTful风格接口使用资源复数作为路径，就无法从路径上看出接口对资源操作
RESTful风格在HTTP请求方法上做文章，约定：

**GET**（SELECT）：从服务器取出资源（一项或多项）。
**POST**（CREATE）：在服务器新建一个资源。
**PUT**（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。PUT更新整个对象 
**PATCH**（UPDATE）：在服务器更新资源（客户端提供改变的属性【补丁】）。 PATCH更新个别属性  
**DELETE**（DELETE）：从服务器删除资源。
**HEAD**：获得一个资源的元数据，比如一个资源的hash值或者最后修改日期； 
**OPTIONS**：获得客户端针对一个资源能够实施的操作(获取该资源的api(能够对资源做什么操作的描述))

***
**示例**

传统方式
```
http://www.xxx.cn/employee/list
http://www.xxx.cn/employee/get?id=1
http://www.xxx.cn/employee/save?name=xx
http://www.xxx.cn/employee/update?id=1&name=xx
http://www.xxx.cn/employee/delete?id=1
```


RESTful方式
http://www.xxx.cn/employees
新增：POST
更新：PUT
删除：DELETE
查询：GET
```
GET /zoos：列出所有动物园 
POST /zoos：新建一个动物园
GET /zoos/{id}：获取某个指定动物园的信息 
PUT /zoos/{id}：更新某个指定动物园的信息（提供该动物园的全部信息）
PATCH /zoos/{id}：更新某个指定动物园的信息（提供该动物园的部分信息）
DELETE /zoos/{id}：删除某个动物园 
GET /zoos/{id}/animals：列出某个指定动物园的所有动物 
```

获取某个部门的所有员工 
```
GET /employee/getByDeptId 传统

GET /departments/{id}/employees  restful风格
```

## 请求参数

由需求决定

## 请求响应

>一般都是返回Json格式数据，具体约束看项目规定

比如
GET /collection：返回资源对象的列表（数组）
GET /collection/resource：返回单个资源对象
POST /collection：返回新生成的资源对象
PUT /collection/resource：返回完整的资源对象
PATCH /collection/resource：返回完整的资源对象
DELETE /collection/resource：返回一个空文档

# HTTP响应状态码

**200** OK - [GET]：服务器成功返回用户请求的数据。
201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
204 NO CONTENT - [DELETE]：用户删除数据成功。

**400** INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
**401** Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
**403** Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
**404** NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。

**500** INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。


# 资源表现形式

文本可以用txt格式表现，也可以用HTML格式、XML格式、JSON格式表现，甚至可以采用二进制格式
图片可以用JPG格式表现，也可以用PNG格式表现，....

资源的具体表现形式，应该在HTTP请求的头信息中用Accept和Content-Type字段指定，这两个字段才是对"表现"的描述。

```
accept:application/json 
content-type:application/json 
```


1.Accept属于请求头， Content-Type属于实体头。 
Http报头分为通用报头，请求报头，响应报头和实体报头。 
请求方的http报头结构：通用报头|请求报头|实体报头 
响应方的http报头结构：通用报头|响应报头|实体报头

2.Accept代表发送端（客户端）希望接受的数据类型。 
比如：Accept：application/json; 
代表客户端希望接受的数据类型是json类型,后台返回json数据

Content-Type代表发送端（客户端|服务器）发送的实体数据的数据类型。
比如：Content-Type：application/json; 
代表发送端发送的数据格式是json, 后台就要以这种格式来接收前端发过来的数据。

![img/accept与content-type.png](https://image.itbaima.net/images/253/image-2023092510320392.png)