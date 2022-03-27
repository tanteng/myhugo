---
title: 深入理解 PHP 中 session 和 cookies 的联系
author: 深 呼吸
type: post
date: 2012-09-26T01:46:43+00:00
url: /2012/09/php-session-cookies/
categories:
  - Develop
  - PHP

---
1. session概念

2. http协议与状态保持

3. 理解cookie

4. php中session的生成机制

5. php中session的过期回收机制

6. php中session的客户端存储机制

<!--more-->

# 1. session概念

在web服务器蓬勃发展的时代，session在web开发语境下的语义是指一类用来在客户端与服务器之间保持状态的解决方案。

# 2. http协议与状态保持

http协议本身是无状态的，客户端只需要简单的向服务器请求下载某些文件，无论是客户端还是服务器都没有必要纪录彼此过去的行为，每一次请求之间都是独立的。

然而人们很快发现如果能够提供一些按需生成的动态信息会使web变得更加有用，就像给有线电视加上点播功能一样。这种需求一方面迫使HTML逐步添加了表单、脚本、DOM等客户端行为，另一方面在服务器端则出现了CGI规范以响应客户端的动态请求，作为传输载体的HTTP协议也添加了文件上载、cookie这些特性。其中cookie的作用就是为了解决HTTP协议无状态的缺陷所作出的努力。至于后来出现的session机制则是又一种在客户端与服务器之间保持状态的解决方案。

session机制可能需要借助于cookie机制来达到保存标识的目的。所以有必要了解下cookie。

# 3. 理解cookie

cookie分发是通过扩展HTTP协议来实现的，服务器通过在HTTP的响应头中加上一行特殊的指示以提示浏览器按照指示生成相应的cookie。然而纯粹的客户端脚本如JavaScript或者VBScript也可以生成cookie。

而cookie 的使用是由浏览器按照一定的原则在后台自动发送给服务器的。浏览器检查所有存储的cookie，如果某个cookie所声明的作用范围大于等于将要请求的资源所在的位置，则把该cookie附在请求资源的HTTP请求头上发送给服务器。

cookie的内容主要包括：名字，值，过期时间，路径和域。

其中域可以指定某一个域比如.google.com，相当于总店招牌，比如宝洁公司，也可以指定一个域下的具体某台机器比如www.google.com或者froogle.google.com，可以用飘柔来做比。路径就是跟在域名后面的URL路径，比如/或者/foo等等，可以用某飘柔专柜做比。

路径与域合在一起就构成了cookie的作用范围。

如果不设置过期时间，则表示这个cookie的生命期为浏览器会话期间，只要关闭浏览器窗口，cookie就消失了。这种生命期为浏览器会话期的 cookie被称为会话cookie。会话cookie一般不存储在硬盘上而是保存在内存里，当然这种行为并不是规范规定的。如果设置了过期时间，浏览器就会把cookie保存到硬盘上，关闭后再次打开浏览器，这些cookie仍然有效直到超过设定的过期时间

存储在硬盘上的cookie 不可以在不同的浏览器间共享，可以在同一浏览器的不同进程间共享，比如两个IE窗口。

这是因为每中浏览器存储cookie的位置不一样，比如

Chrome下的cookie放在：

C:\Users\sharexie\AppData\Local\Google\Chrome\User Data\Default\Cache

Firefox下的cookie放在：

C:\Users\sharexie\AppData\Roaming\Mozilla\Firefox\Profiles\tq2hit6m.default\cookies.sqlite （倒数第二个文件名是随机的文件名字）

Ie下的cookie放在：

C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Cookies

（网上都说是在这里，但是我一直没找到）

我在这里也有一个测试，在firefox下用httplook软件进行嗅探：

1、在本机上第一次打开必应网站，抓包：

![][1]

返回的数据如下：

HTTP/1.1 200 OK

Cache-Control: private, max-age=0

Content-Type: text/html; charset=utf-8

Content-Encoding: gzip

Set-Cookie: _FS=NU=1; domain=.bing.com; path=/

Set-Cookie: _SS=SID=442E36ABF8F5431E8DFF0CAC018437E3; domain=.bing.com; path=/

Set-Cookie: MUID=32B1FE9DB0EB65B52006FD50B1E86565; expires=Sun, 31-Aug-2014 11:35:51 GMT; domain=.bing.com; path=/

Set-Cookie: OrigMUID=32B1FE9DB0EB65B52006FD50B1E86565%2c15deb35b84a74788ae2d9978e3e657b1; expires=Sun, 31-Aug-2014 11:35:51 GMT; domain=.bing.com; path=/

Set-Cookie: SRCHD=D=2454455&MS=2454455&AF=NOFORM; expires=Sun, 31-Aug-2014 11:35:51 GMT; domain=.bing.com; path=/

Set-Cookie: SRCHUID=V=2&GUID=F6DCC04B2CC54139928925763DAEE04A; expires=Sun, 31-Aug-2014 11:35:51 GMT; path=/

Set-Cookie: SRCHUSR=AUTOREDIR=0&GEOVAR=&DOB=20120831; expires=Sun, 31-Aug-2014 11:35:51 GMT; domain=.bing.com; path=/

P3P: CP=&#8221;NON UNI COM NAV STA LOC CURa DEVa PSAa PSDa OUR IND&#8221;

Date: Fri, 31 Aug 2012 11:35:50 GMT

Content-Length: 12787

X-Cache-Lookup: MISS from proxy:8080

我们可以看到sessionId为442E36ABF8F5431E8DFF0CAC018437E3，domain为.bing.com; path为/。服务器为本用户建立一个session，id为442E36ABF8F5431E8DFF0CAC018437E3作为客服端的cookie中的SID的值。

2、第二次请求必应网站，请求内容如下：

![][2]

看到请求中带有sid为442E36ABF8F5431E8DFF0CAC018437E3的cookie。

服务器返回数据：

HTTP/1.1 200 OK

Cache-Control: private, max-age=0

Content-Type: text/html; charset=utf-8

Content-Encoding: gzip

P3P: CP=&#8221;NON UNI COM NAV STA LOC CURa DEVa PSAa PSDa OUR IND&#8221;

Date: Fri, 31 Aug 2012 11:41:12 GMT

Content-Length: 12437

X-Cache-Lookup: MISS from proxy:8080

服务器查看本tmp目录中有一个文件的名字和SID匹配，知道是一个老用户，没有新建session，直接返回数据。

当然也有很多304的返回，表示在expires内直接用用户的缓存即可。

![][3]

# 4. php中session的生成机制

我们先来分析一下PHP中是怎么生成一个session的。设计出session的目的是保持每一个用户的各种状态来弥补HTTP协议的不足(无状态)。session是保存在服务器的，既然它用于保持每一个用户的状态那它利用什么来区别用户的呢？这个时候就得借助cookie了。当我们在代码中调用session_start();时，PHP会同时往SESSION的存放目录(默认为/tmp/)和客户端的cookie目录各生成一个文件。session文件名称像这样：

![][4]

格式为sess\_{SESSIONID} ，这时session文件中没有任何内容，当我们在session\_start();添加了这两行代码：

$_SESSION[&#8216;name&#8217;] =&#8217;sharexie&#8217;;

$_SESSION[&#8216;webUlr&#8217;] = &#8216;www.qq.com&#8217;;

这时文件就有内容了：

name|s:8:&#8221;sharexie&#8221;;webUlr|s:10:&#8221;www.qq.com&#8221;;

这时再看看cookie：

![][5]

可以看到服务器为我们自动生成了一个cookie,cookie名称为&#8221;PHPSESSID&#8221;，cookie内容是一串字符，其实这串字符就是{SESSIONID}。当我们使用session时，PHP就先生成一个唯一的SESSIONID号(如2bd170b3f86523f1b1b60b55ffde0f66)，再在我们服务器的默认目录下生成一个文件，文件名为sess_{SESSIONID}，同时在当前用户的客户端生成一个cookie，内容已经说过了。这样PHP会为每一个用户生成一个SESSIONID，也就是说一个用户一个session文件。PHP第一次为某个用户使用session时就向客户端写入了cookie，当这个用户以后访问时，浏览器会带上这个cookie，PHP在拿到cookie后就读出里面的SESSIONID，拿着这个SESSIONID去session目录下找session文件。

# 5. php中session的过期回收机制

我们明白了session的生成及工作原理，发现在session目录下会有许多session文件。当然这些文件一定不是永远存在的，PHP一定提供了一种过期回收机制。在php.ini中session.gc_maxlifetime为session设置了生存时间(默认为1440s)。如果session文件的最后更新时间到现在超过了生存时间，这个session文件就被认为是过期的了。在下一次session回收的时候就会被删除。那下一次session回收是在什么时候呢？这和php请求次数有关的。在PHP内部机制中，当php被请求了N次后就会有一次触发回收机制。到底是请求多少次触发一次是通过以下两个参数控制的：

session.gc_probability = 1

session.gc_divisor = 100

这是php.ini的默认设置，意思是每100次PHP请求就有一次回收发生。概率是gc\_probability/gc\_divisor (这里我把session.gc_divisor改为1，好像访问了很多次都没有触发回收事件，不知道什么原因)。我们了解了服务器端的session过期机制，再来看看客户端的cookie的过期机制。

如果cookie失效了浏览器自然发送不了cookie到服务器，这时即使服务器的session文件存在也没用，因为PHP不知道要读取哪个session文件。我们知道PHP的cookie过期时间是在创建时设置的，那么PHP在创建session的同时为客户端创建的cookie的生命周期是多久呢？这个在php.ini中有设置：session.cookie\_lifetime 。这个值默认是0，代表浏览器一关闭SESSIONID就失效。那就是说我们把session.gc\_maxlifetime和session.cookie_lifetime设置成同一个值就可以控制session的失效时间了。

# 6. php中session的客户端存储机制

由于cookie可以被人为的禁止，必须有其他机制以便在cookie被禁止时仍然能够把session id传递回服务器。解决办法有：

1、URL重写，就是把session id直接附加在URL路径的后面，一种是作为URL路径的附加信息，表现形式为http://&#8230;../xxx;jsessionid= ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764

2、另一种是作为查询字符串附加在URL后面，表现形式为http://&#8230;../xxx?jsessionid=ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764
  
这两种方式对于用户来说是没有区别的，只是服务器在解析的时候处理的方式不同，采用第一种方式也有利于把session id的信息和正常程序参数区分开来。

为了在整个交互过程中始终保持状态，就必须在每个客户端可能请求的路径后面都包含这个session id。

3、表单隐藏字段。就是服务器会自动修改表单，添加一个隐藏字段，以便在表单提交时能够把session id传递回服务器。比如下面的表单
  
<form name=&#8221;testform&#8221; action=&#8221;/xxx&#8221;>
  
<input type=&#8221;text&#8221;>
  
</form>
  
在被传递给客户端之前将被改写成
  
<form name=&#8221;testform&#8221; action=&#8221;/xxx&#8221;>
  
<input type=&#8221;hidden&#8221; name=&#8221;jsessionid&#8221; value=&#8221;ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764&#8243;>
  
<input type=&#8221;text&#8221;>
  
</form>
  
实际上这种技术可以简单的用对action应用URL重写来代替。

<audio style="display: none;" controls="controls"></audio>

<audio style="display: none;" controls="controls"></audio>

 [1]: http://pic002.cnblogs.com/images/2012/336475/2012083122350328.png
 [2]: http://pic002.cnblogs.com/images/2012/336475/2012083122351610.png
 [3]: http://pic002.cnblogs.com/images/2012/336475/2012083122362012.png
 [4]: http://pic002.cnblogs.com/images/2012/336475/2012083122363460.png
 [5]: http://pic002.cnblogs.com/images/2012/336475/2012083122364519.png