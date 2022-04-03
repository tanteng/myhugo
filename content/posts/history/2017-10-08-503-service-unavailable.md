---
title: 服务器返回 502,503,504 原因及解决办法
author: 深 呼吸
type: post
date: 2017-10-08T14:44:08+00:00
url: /2017/10/503-service-unavailable/
categories:
  - Develop
  - Nginx

---
### 502 bad gateway

问题原因：服务器作为网关或代理，从上游服务器收到无效响应。

nginx 在这里充当的是反向代理服务器的角色，是把http协议请求转成fastcgi协议的请求，通过 fastcgi_pass指令传递给 php-fpm进程，当 php-fpm 进程响应的内容是 nginx 无法理解的响应，就会返回 502 bad gateway

<!--more-->

解释：出现502错误，通常意味着一两个机器已经不正确，简单点说，就是机器挂掉了。理论点儿说，nginx执行请求的时候，却收到了上游服务器的无效响应

### 503 service unavailable

灾难事件：临时的服务器维护/过载，服务器当前无法处理请求，报503

问题原因：

1.请求用户量太多，服务器为了保护自己不挂掉，机智的拒绝某些用户的访问，这些用户就会收到 503 这个错误.

2.以 PHP 作为后端语言的话，一个http请求占用一个php-fpm进程，瞬时请求量过大时，没有足够的php-fpm进程去处理请求，就会返回 503 service unavailable

解决办法： 等一会儿仔访问该网站或者尝试强刷新页面，问题一般就能够解决了。

### 504 请求超时

事件描述：dns 查询过程超时，返回 504；摸不着头脑，不管访问什么网站，都报504 这个错误

如果是 PHP，单个 php-fpm进程阻塞超过nginx的时间阈值返回504 gateway timeout

问题原因：nginx 或者后端配置不正确

解决办法：nginx 或后端的配置参数设置正确合理

解释：实际上 504 很少会遇到，通常这个错误是由于 nginx 配置不当引起的，比如你将你的nginx的超时时间设置为 300，那么如果此次请求的响应时间超过了 300，你就会看到 504 这个报错。