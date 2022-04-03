---
title: Composer的自动加载机制
author: 深 呼吸
type: post
date: 2015-12-24T15:41:16+00:00
url: /2015/12/composer-autoload/
duoshuo_thread_id:
  - 6231875631647294209
categories:
  - Develop
  - PHP
format: link

---
如项目下的composer.json文件中有autoload的定义：

<code class="lang:php decode:true ">"autoload": {
        "classmap": [
            "database"
        ],
        "psr-4": {
            "GrahamCampbell\\BootstrapCMS\\": "app/"
        }
    },</code>

这样定义如何实现自动加载呢？这里classmap和psr-4的区别又是什么？

<!--more-->

了解PHP依赖包管理工具Composer的自动加载机制，链接：

<a href="http://www.tuicool.com/articles/mARrMj6" target="_blank" rel="nofollow">http://www.tuicool.com/articles/mARrMj6</a>