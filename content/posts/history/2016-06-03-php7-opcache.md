---
title: PHP 7安装和开启opcache
author: 小谈
type: post
date: 2016-06-02T16:20:20+00:00
url: /2016/06/php7-opcache/
duoshuo_thread_id:
  - 6291630689007698689
categories:
  - Develop
  - PHP

---
鸟哥在博客中说，提高PHP 7性能的几个tips，第一条就是开启opache，引用下原文：

> 记得启用Zend Opcache, 因为PHP7即使不启用Opcache速度也比PHP-5.6启用了Opcache快, 所以之前测试时期就发生了有人一直没有启用Opcache的事情

<!--more-->

我的阿里云服务器是通过一个叫remi的centos源提供的PHP 7，默认没装opcache，用yum list yum70*命令搜索一下，果然有，于是install下来：

yum install php70-php-opcache.x86_64

然后重启php-fpm，使用service php70-php-fpm restart命令。（不同的系统和环境根据自己的情况安装和开启opache）

新建一个php文件，用phpinfo()函数显示php.ini信息，如果可以找到OPcache项，说明成功启用了。安装和启用PHP 7，就是给PHP 7再加一个翅膀！