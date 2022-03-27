---
title: Nginx 报错和解决方法记录
author: 深 呼吸
type: post
date: 2017-04-19T07:49:24+00:00
url: /2017/04/nginx-permission-denied/
categories:
  - Develop
  - PHP

---
记录一下遇到的各种 Nginx 的报错和解决办法。

### 13: Permission denied

Nginx错误：

2017/04/19 14:46:46 [crit] 4172#0: *671 open() &#8220;/data/vhosts/xunlei.com/test/&#8221; failed (13: Permission denied), client: 192.168.35.54, server: www.test.com, request: &#8220;GET / HTTP/1.1&#8221;, host: &#8220;www.test.com&#8221;

经查权限问题导致，网站目录是 root 用户组，而 nginx 是运行的 nobody 用户进程，修改网站目录为 nobody 用户组。

<!--more-->

### 111: Connection refused

Nginx错误：

2017/04/19 14:48:02 [error] 4172#0: *672 connect() failed (111: Connection refused) while connecting to upstream, client: 192.168.35.54, server: www.test.com, request: &#8220;GET / HTTP/1.1&#8221;, upstream: &#8220;fastcgi://127.0.0.1:9000&#8221;, host: &#8220;www.test.com&#8221;

**出现的原因：**

1.php-fpm 服务没有开启

2.权限问题，查看目录用户组

### *1111335 upstream prematurely closed connection

Nginx错误：

2017/04/28 17:34:48 [error] 11058#0: *1111335 upstream prematurely closed connection while reading response header from upstream, client: 192.168.35.54, server: www.test.com, request: &#8220;GET /test/TestSADD HTTP/1.1&#8221;, upstream: &#8220;http://127.0.0.1:8080/test/TestSADD&#8221;, host: &#8220;www.test.com&#8221;

出现错误的背景是使用 golang 的一个项目在 logs 目录下 debug.log 写日志，报 502 错误。

在业务代码打印错误日志：open logs/debug.log: no such file or directory

这个就归类于代码问题导致 nginx 挂掉，解决办法：修改业务代码。

### 110: Connection timed out

18244#0: *7617 upstream timed out (110: Connection timed out) while reading response header from upstream, client: 192.168.35.54, server: eagleye.vip.xx.com, request: &#8220;POST /rules HTTP/1.1&#8221;, upstream: &#8220;fastcgi://127.0.0.1:9000&#8221;, host: &#8220;eagleye.vip.xx.com&#8221;, referrer: &#8220;http://eagleye.vip.xx.com/rules/create?type=2&#8221;

错误背景：循环分批写入 uids （用户id）到 Redis 集合，文件过大超时。