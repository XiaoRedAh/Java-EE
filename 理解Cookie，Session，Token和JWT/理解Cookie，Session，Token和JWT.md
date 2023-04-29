# 前后端分离不等同于抛弃Session

前后端分离并不等同于抛弃session。

在实际开发中，很多前后端分离的项目都不再采用传统的基于session的身份认证方式。这是因为前后端分离的架构中，前端和后端是独立的两个模块，前端通过API接口来获取数据和操作资源，而这些操作都需要身份认证和权限控制。基于session的身份认证在前后端分离的架构中存在许多问题，比如：

1. 维护session状态的复杂性：前端和后端是独立的两个模块，需要通过API接口进行通讯。使用基于session的身份认证，前端需要在每个请求中都携带session ID，这会增加很多额外的网络开销和代码复杂性。
2. 前后端耦合度高：传统的基于session的身份认证要求前后端共享session状态，这会增加耦合度，限制前端和后端的独立性和可移植性。
3. 扩展性差：基于session的身份认证难以支持分布式系统和高并发场景，会影响系统的扩展性和性能表现。

因此，在前后端分离的架构中，通常会采用基于Token的身份认证方式，如JWT（JSON Web Token），来代替传统的基于session的身份认证方式。Token是一种轻量级的身份认证方案，只包含必要的用户信息和签名，前端通过API接口将Token发送给后端进行身份认证和权限控制，这种方式解决了上述问题，提高了系统的可移植性和扩展性。

# 如何取舍

Session：简单的单体网站

Token，JWT：系统同时登录用户多，集群服务器多，有单点登录需求，分布式部署。

# 概述

Cookie：是存储在客户端浏览器中的小文本文件，可以在网站之间传递数据。Cookie的主要特点是可以设置过期时间和域名访问限制。

Session：是存储在服务器端的会话信息，与每个用户绑定。在服务器端创建一个Session ID，通过Cookie将Session ID传回给客户端，在客户端提交请求时再将Session ID传回服务器端来维持会话。Session的主要特点是安全，数据存储在服务器，客户端无法直接访问。

Token：是以JSON格式为载体的一串字符串，存储在客户端，并经过加密处理，用来传递认证信息。与Cookie相比，Token不需要在客户端保存任何状态信息，服务器也不需要保存任何用户信息，只需要将Token发送给服务器进行验证即可。

JWT：是一种基于Token实现的身份认证解决方案，也是一种开放标准（RFC 7519）。JWT通过将用户信息编码成一段JSON数据并使用签名（用于验证数据完整性和真实性）生成一个Token，来实现用户身份认证和数据传输。

区别：

* Cookie和Session存储位置不同，Cookie存储在客户端，Session存储在服务器端。
* Token和JWT都是存储在客户端的认证信息，但Token没有JWT那么灵活和安全。
* JWT不需要在服务器端保存用户信息，相比于Session更加轻量级。

# Cookie

Cookie由服务器（后端）生成，然后发送并保存到用户浏览器。浏览器下次向同一服务器发起请求时会携带这些信息

![](img/%E5%9B%BE%E7%A4%BACookie%E6%9C%BA%E5%88%B6.png)

一个Cookie包含的信息：

* **name** - Cookie的名称，Cookie一旦创建，名称便不可更改
* **value** - Cookie的值，如果值为Unicode字符，需要为字符编码。如果为二进制数据，则需要使用BASE64编码
* **maxAge** - Cookie失效的时间，单位秒。如果为正数，则该Cookie在maxAge秒后失效。如果为负数，该Cookie为临时Cookie，关闭浏览器即失效，浏览器也不会以任何形式保存该Cookie。如果为0，表示删除该Cookie。默认为-1。
* secure - 该Cookie是否仅被使用安全协议传输。安全协议。安全协议有HTTPS，SSL等，在网络上传输数据之前先将数据加密。默认为false。
* path - Cookie的使用路径。如果设置为“/sessionWeb/”，则只有contextPath为“/sessionWeb”的程序可以访问该Cookie。如果设置为“/”，则本域名下contextPath都可以访问该Cookie。注意最后一个字符必须为“/”。
* **domain** - 可以访问该Cookie的域名。如果设置为“.google.com”，则所有以“google.com”结尾的域名都可以访问该Cookie。注意第一个字符必须为“.”。
* comment - 该Cookie的用处说明，浏览器显示Cookie信息的时候显示该说明。
* version - Cookie使用的版本号。0表示遵循Netscape的Cookie规范，1表示遵循W3C的RFC 2109规范

Cookie是不可跨域的： 每个Cookie都会绑定单一的域名（绑定域名下的子域都是有效的），无法在别的域名下获取使用，同域名不同端口也是允许共享使用的。

# Session

每个用户的会话都会有一个自己的Session对象，来自同一个浏览器的所有请求，都属于同一个会话。这样的话，通过Session，就可以在无连接的HTTP中辨别当前请求是来自哪个用户的。

Session实际上是**基于Cookie实现**的：

1. 服务器（后端）创建Session
2. 将这个Session的id存放在客户端（服务器）中一个叫JSESSIONID的Cookie中。

![](img/%E5%9B%BE%E7%A4%BASession%E6%9C%BA%E5%88%B6.png)

只要浏览器携带此Cookie访问服务器，服务器就会通过Cookie的值进行辨别，得到对应的Session对象。因此，这样就可以追踪到底是哪一个浏览器在访问服务器。

由于Session中的信息（如用户信息）是存放在后端的，浏览器的Cookie存放的是仅仅Session的id，因此安全性比直接在Cookie中存用户信息高很多。

Session并不是永远都存在的，默认时间为30分钟，若超过此时间，Session将丢失

**基于服务器验证方式的弊端**

1. 在服务端的内存中使用Seesion存储登录信息，当越来越多的用户发请求时，内存的开销也会不断增加。每次请式都需要用户向已验证的服务器发送验证信息(Session亲和性),用户量大时，可能会造成
些拥堵
1. 可扩展性不好
2. CORS跨域问题
3. CSRF跨站请求伪造

# Token

基于Token的身份验证是**无状态**的，不将用户信息存在服务器或Session中。这种概念解决了在服务端存储信息时的许多问题

**Token令牌构成**

* uid：用户唯一身份标识
* timestamp：当前时间戳
* sign：签名字符串，防止第三方伪造数据；签名密钥是存储在服务器端的，其它人无法知道
* 其它附加参数。

**基于Token的身份验证的过程如下**

1. 用户通过用户名和密码发送请求
2. 程序验证。
3. 程序生成并返回一个token给客户端
4. 客户端储存token,并且每次请求都携带token
5. 服务端验证请求中携带的token，验证成功才并返回数据。

每一次请求都需要Token。Token应该在HTTP的头部发送从而保证HTTP请求无状态

![](img/%E5%9B%BE%E7%A4%BAToken%E6%9C%BA%E5%88%B6.png)

**Token优势**

1. 在客户端存储的Tokens是无状态的，并且能够被扩展。基于这种无状态和不存储Session信息，负载负载均衡器能够将用户信息从一个服务传到其他服务器上。
2. 使用tokens之后这些问题都迎刃而解，因为tokens自己hold住了用户的验证信息。
3. 安全性
   * 请求中发送token而不再是发送cookie能够**防止CSRF跨站请求伪造**。即使在客户端使用cookie存储token，cookie也仅仅是一个存储机制而不是用于认证。
   * 不将信息存储在Session中，减少对Session的操作。
   * token是有时效的，一段时间之后用户需要重新验证。也可以通过tokerrevocataion主动把token撤回
4. 可扩展性
tokens能够创建**与其它程序共享权限**的程序。使用tokens时，可以提供可选的权限给第三方应用程序。当用户想让另一个应用程序访问它们的数据，可以通过建立自己的API，得出特殊权限tokens。
5. 多平台跨域
只要用户有一个通过了验证的token，数据和资源就能够在任何域上被请求到。

# JWT

JSON Web Token（简称 JWT）是为了在网络应用环境间传递声明而执行的一种基于 JSON 的开放标准（RFC 7519）。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源。比如用在用户登录上。

可以使用 HMAC 算法或者是 RSA 的公/私秘钥对 JWT 进行签名。因为数字签名的存在，这些传递的信息是可信的。

**Web Token构成**

* Header：{ "alg": "HS256", "typ": "JWT" } 
    * type：代表token的类型，这里使用的是JWT类型。 
    * alg:使用的Hash算法，例如HMAC SHA256或RSA。
* Payload：荷载信息，包含一些声明Claim(实体的描述，通常是一个User信息，还包括一些其他的元数据) 
    * Reserved Claims：这是一套预定义的声明，并不是必须的。包括：iss(issuer)、exp(expiration time)、sub(subject)、aud(audience)等 
    * Plubic Claims
    * Private Claims，交换信息的双方自定义的声明 { "sub": "1234567890", "name": "John Doe", "admin": true } 同
* signature：用来确认JWT信息的发送者是谁，并保证信息没有被修改
使用header中指定的算法将编码后的header、编码后的payload、一个secret进行加密。 
例如使用的是HMAC SHA256算法，大致流程类似于: HMACSHA256( base64UrlEncode(header) + "." + base64UrlEncode(payload), secret) 

**JWT和Token**

相同：

* 都是认证授权机制（也可以说都是令牌），只有验证成功后，客户端才能访问服务端上受保护的资源
* 都可以记录用户的信息
* 都是在服务端无状态化的认证机制，因为用户的状态不再存储在服务端的内存中

区别：

1. 得出token算法不一样
    token包含： uid（用户唯一身份标识）+  timestamp（当前时间戳） + sign（签名）
    WT包含：header（头部） + payload（负载 ）+ sign（签名）

2. 在客户端存储位置
    Token一般存储在Cookie中。JWT一般不使用Cookie，而是放在请求头 Authorization中。
    因为JWT不使用Cookie，所以可以使用任何域名提供你的API 服务而不需要担心跨域资源共享问题（CORS）

3. Token需要查询数据库，JWT不用
    Token：服务端验证客户端发送过来的Token时，还需要查询数据库获取用户信息，然后验证Token是否有效。
    JWT：将Token和Payload加密后存储于客户端，服务端只需要使用密钥解密进行校验（校验也是 JWT 自己实现的）即可，不需要查询或者减少查询数据库，因为 JWT 自包含了用户信息和加密的数据。




