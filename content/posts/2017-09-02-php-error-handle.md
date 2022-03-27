---
title: PHP 错误和异常处理
author: 深 呼吸
type: post
date: 2017-09-02T07:27:24+00:00
url: /2017/09/php-error-handle/
categories:
  - Develop
  - PHP

---
关于 PHP 的错误和异常处理，总结如下。

### 1.设置 PHP 错误级别

使用 <span class="refname">error_reporting</span> — <span class="dc-title">设置应该报告何种 PHP 错误。</span>

<!--more-->

常见范例：

<code>&lt;?php

// 关闭所有PHP错误报告
error_reporting(0);

// Report simple running errors
error_reporting(E_ERROR | E_WARNING | E_PARSE);

// 报告 E_NOTICE也挺好 (报告未初始化的变量或者捕获变量名的错误拼写)
error_reporting(E_ERROR | E_WARNING | E_PARSE | E_NOTICE);

// 除了 E_NOTICE，报告其他所有错误
error_reporting(E_ALL ^ E_NOTICE);

// 报告所有 PHP 错误 (参见 changelog)
error_reporting(E_ALL);

// 报告所有 PHP 错误
error_reporting(-1);

// 和 error_reporting(E_ALL); 一样
ini_set('error_reporting', E_ALL);
</code>

### 2.设置自定义错误处理函数

<span class="refname">set_error_handler</span> — <span class="dc-title">设置用户自定义的错误处理函数</span>

### 3.设置 PHP 运行结束的处理

register\_shutdown\_function 设置致命错误的处理，如程序异常，超时等，在 PHP 程序运行结束时触发。

### 参考链接

<a href="http://php.net/manual/zh/book.errorfunc.php" target="_blank" rel="noopener nofollow">http://php.net/manual/zh/book.errorfunc.php</a>