---
title: PHP知识整理之——PHP基础、WEB安全、网络
author: 深 呼吸
type: post
date: 2018-04-10T03:11:14+00:00
url: /2018/04/php-2/
categories:
  - Develop
  - PHP

---
本篇文章是PHP知识系统整理系列之——PHP基础、WEB安全、网络，包括 PSR 规范，PHP7特性和性能提升，HTTP、HTTPS、TCP、WebSocket协议，WEB安全和计算机网络的内容。以及 PHP session 回收，php-fpm 调优，HTTP/2 等知识点。<!--more-->

## PHP 基础部分

### PSR标准规范

PSR 是 PHP Standard Recommendations 的简写，由 PHP FIG 组织制定
  
的 PHP 规范，是 PHP 开发的实践标准。

PHP FIG， FIG 是 Framework Interoperability Group（框架可互用性小组）的缩写，由几位开源框架的开发者成立于 2009 年，从那开始也选取了很多其他成员进来（包括但不限于 Laravel, Joomla, Drupal, Composer, Phalcon, Slim, Symfony, Zend Framework 等），虽然不是「官方」组织，但也代表了大部分的 PHP 社区。

项目的目的在于：通过框架作者或者框架的代表之间讨论，以最低程度的限制，制定一个协作标准，各个框架遵循统一的编码规范，避免各家自行发展的风格阻碍了 PHP 的发展，解决这个程序设计师由来已久的困扰。

目前已表决通过了 6 套标准，已经得到大部分 PHP 框架的支持和认可。

<table>
  <tr>
    <td width="477">
      PSR 1
    </td>
    
    <td width="550">
      基础编码规范
    </td>
  </tr>
  
  <tr>
    <td width="477">
      PSR 2
    </td>
    
    <td width="550">
      编程风格规范
    </td>
  </tr>
  
  <tr>
    <td width="477">
      PSR 3
    </td>
    
    <td width="550">
      日志接口规范
    </td>
  </tr>
  
  <tr>
    <td width="477">
      PSR 4
    </td>
    
    <td width="550">
      自动加载规范
    </td>
  </tr>
  
  <tr>
    <td width="477">
      PSR 6
    </td>
    
    <td width="550">
      缓存接口规范
    </td>
  </tr>
  
  <tr>
    <td width="477">
      PSR 7
    </td>
    
    <td width="550">
      HTTP消息接口规范
    </td>
  </tr>
</table>

注： PSR 0 自动加载已废弃， PSR 4取代， PSR 5 PHP doc规范还在起草，以及PSR 8， 9， 10， 11， 12， 13

### PHP 7 新特性

  * 标量和返回值类型声明
  * null 合并运算符、组合比较符、匿名类等新语法
  * AST（抽象语法树）
  * 性能突破
  * Zend引擎优化
  * 函数调用机制

#### 标量和返回值类型声明

如图，函数变量和返回值都可以声明类型。

#### 抽象语法树

在计算机科学中，抽象语法树（ Abstract Syntax Tree， AST），或简称语法树（ Syntax tree），是源代码语法结构的一种抽象表示。 它以树状的形式表现编程语言的语法结构，树上的每个节点都表示源代码中的一种结构。

PHP Opcode 是一种 PHP 脚本编译后的中间语言，就像Java 的 ByteCode 或者 .NET 的MSL.

PHP 7 中新增了抽象语法树，代码的解释执行过程变化如图所示：

AST 在 PHP 编译过程作为一个中间件的角色，替换原来直接从解释器吐出 opcode 的方式，让解释器（parser）和编译器（compliler）解耦，可以减少一些Hack代码，同时，让实现更容易理解和可维护。

### PHP-FPM调优

PHP_FPM 是 PHP 的 FastCGI 的进程管理器。

CGI 是 fork-and-execute 方式，每次请求都需要创建进程，每个进程都需要解析 php.ini 配置，连接服务器，逻辑处理和退出过程，效率低下。

Fast-CGI 是 CGI 增强版， WEB 服务器启动时同时启动 PHP 进程管理器， 1 个 master 进程和N 个子进程（ worker 进程），可以动态增减子进程。

pm = dynamic //static
  
pm.max_children = 8
  
pm.start_servers = 2
  
pm.min\_spare\_servers = 2
  
pm.max\_spare\_servers = 4
  
pm.max_requests = 1000

### PHP Session 回收参数设置

php.ini 中 session gc 设置：

session.save_handler = files
  
session.gc_probability =1
  
session.gc_divisor =1000
  
session.gc_maxlifetime =1440//过期时间 默认24分钟

session 回收执行的概率是 session.gc\_probability/session.gc\_divisor

结果是 1/1000

## WEB安全

WEB安全涉及到的内容：

  * 密码哈希
  * XSS攻击
  * CSRF攻击
  * SQL注入（ PDO原理）
  * 对称加密、非对称加密
  * Hash算法

### 密码哈希

过去对密码进行 md5， sha1 等方式，加 salt 加密后保存在数据
  
库，如今这种方式已经不再安全。

哈希算法：哈希算法是不可逆的，加密算法是可逆的。

常见的哈希算法： MD2、 MD4、 MD5、 SHA 等

对称加密算法： DES、 3DES、 AES、 Blowfish、 IDEA、 RC4、
  
RC5、 RC6

现在更安全的方式是使用 password\_hash 生成密码，用 password\_verify 验证密码，还有更多选项。

### XSS攻击

按照类型划分：

1.非持久型攻击，仅对当前页面造成影响

2.持久型攻击，存储在数据中，可能重复造成影响

也可以分为三类：

1.反射型，经过后端，不经过数据库

2.存储型，经过后端，经过数据库

3.DOM，不经过后端，如通过 url 参数触发

#### XSS攻击原理和防范

原理：

通过 url，表单提交等入口，将脚本提交，触发页面执行脚本，从而可以修改页面，截取 cookie 获取用户信息等。

防范：

不相信用户的任何输入，对输入做过滤，转义处理。同时也可一定程度避免 sql 注入。

### CSRF攻击

跨站请求伪造，是一种挟持用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。

#### CSRF攻击原理和防范

  * 合理使用 GET 和 POST 请求， GET 用于获取资源， POST 用于创建资源。
  * 服务器端每次请求生成一个 token，放在表单隐藏域，同时存到 session 中，每次提交请求进行 token 的验证，防止第三方伪造。 (这也是 Laravel 框架的方案)

#### SQL注入和防范

  * 过滤、转义参数
  * 使用 PDO 操作数据库

#### PDO原理

  * 一种是 PHP 开启了模拟预处理，实际上只是对参数进行了转义，可以防范大部分 sql 注⼊攻击。
  * 一种是 PHP 关闭了模拟与处理，那么 sql 和参数是分两次提交，由 mysql server对 sql 进行预处理，从而防止 sql 注入。（第二种可以完全杜绝 sql 注入攻击）

在一个应用中，打开 mysql 服务器的 sql 日志，也可以看到通过 PDO 方式执行 sql，分为两次提交，第一次是 prepare，第二次 excute，如图。

## 计算机网络

### 关于HTTP协议

  * 常见状态码
  * HTTP动词
  * 无状态
  * keep-alive
  * 协议头
  * cookie 和 session 区别

### HTTP 2.0 特点

二进制分帧层。 HTTP/2 所有性能增强的核⼼在于新的二进制分帧层，它定义了如何封装 HTTP 消息并在客户端与服务器之间传输。

多路复用。同一个连接可同时发起多重请求-响应消息。应用层和传输层之间增加了一个二进制分帧层： HEADERFrame、 Data Frame

标头压缩。 HEADER Compression

服务器推送。 Server Push

### TCP三次握手和四次挥手

图片缺失

#### 为什么需要三次握手而不是两次？

为了防止已失效的连接请求报⽂段突然又传送到了服务端，从而发生错误。失效的报文抵达了服务端，由于只有两次握手，服务端进⼊ESTABLISHED状态等待客户端发送数据，但客户端已经进入CLOSED状态。

具体说明

在谢希仁著《计算机网络》第四版中讲“三次握手”的目的是“为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误”。在另一部经典的《计算机网络》一书中讲“三次握手”的目的是为了解决“网络中存在延迟的重复分组”的问题。这两种不用的表述其实阐明的是同一个问题。

谢希仁版《计算机网络》中的例子是这样的，“已失效的连接请求报文段”的产生在这样一种情况下：client发出的第一个连接请求报文段并没有丢失，而是在某个网络结点长时间的滞留了，以致延误到连接释放以后的某个时间才到达server。本来这是一个早已失效的报文段。但server收到此失效的连接请求报文段后，就误认为是client再次发出的一个新的连接请求。于是就向client发出确认报文段，同意建立连接。假设不采用“三次握手”，那么只要server发出确认，新的连接就建立了。由于现在client并没有发出建立连接的请求，因此不会理睬server的确认，也不会向server发送数据。但server却以为新的运输连接已经建立，并一直等待client发来数据。这样，server的很多资源就白白浪费掉了。采用“三次握手”的办法可以防止上述现象发生。例如刚才那种情况，client不会向server的确认发出确认。server由于收不到确认，就知道client并没有要求建立连接。”

#### 为什么A要先进入TIME-WAIT状态，等待2MSL时间后才进入CLOSED状态

为了保证服务器能收到客户端的确认应答。若客户端发完确认应答后直接进入CLOSED状态，那么如果该应答丢失， 服务器等待超时后就会重新发送连接释放请求，但此时客户端已经关闭了，不会作出任何响应，因此服务器永远无法正常关闭。

### HTTPS连接过程

1.浏览器将自己支持的一套加密规则发送给网站。

2.网站从中选出一组加密算法与HASH算法，并将自己的身份信息以证书的形式发回给浏览器。证书里面包含了网站地址，加密公钥，以及证书的颁发机构等信息。

3.浏览器获得网站证书之后浏览器要做以下工作：

a) 验证证书的合法性（颁发证书的机构是否合法，证书中包含的网站地址是否与正在访问的地址一致等），如果证书受信任，则浏览器栏⾥⾯会显示一个小锁头，否则会给出证书不受信的提示。

b) 如果证书受信任，或者是用户接受了不受信的证书，浏览器会⽣成一串随机数的密码，并用证书中提供的公钥加密。

c) 使用约定好的HASH算法计算握手消息，并使用生成的随机数对消息进⾏加密，最后将之前生成的所有信息发送给网站。

4.网站接收浏览器发来的数据之后要做以下的操作：

a) 使用自己的私钥将信息解密取出密码，使用密码解密浏览器发来的握手消息，并验证HASH是否与浏览器发来的一致。

b) 使用密码加密一段握手消息，发送给浏览器。

5.浏览器解密并计算握手消息的HASH，如果与服务端发来的HASH一致，此时握手过程结束，之后所有的通信数据将由之前浏览器生成的随机密码并利用对称加密算法进行加密。

### WebSocket连接过程

WebSocket 是一个持久化协议， WebSocket 是基于 HTTP 协议的，或者说借用 HTTP的协议来完成一部分握手。

1、客户端发送 GET 请求，请求 ws 协议的地址，同时通过 header 头部告诉服务端要进行协议升级

```http request
GET ws://localhost:3000/ws/chat HTTP/1.1
Host: localhost
Upgrade: websocket
Connection: Upgrade
Origin: http://localhost:3000
Sec-WebSocket-Key: client-random-string
Sec-WebSocket-Version: 13
```

2、服务器给客户端响应，即建立了连接

```http request
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: server-random-string
```

WebSocket 连接建立后，跟 http 就没关系了。

链接：

<a href="https://blog.tanteng.me/2017/07/php-examination-part-1/" target="_blank" rel="noopener">PHP 笔试面试题精选（一）</a>



<audio style="display: none;" controls="controls"></audio>

<audio style="display: none;" controls="controls"></audio>

<audio style="display: none;" controls="controls"></audio>

<audio style="display: none;" controls="controls"></audio>