---
title: PHP intval 转换浮点数精度丢失问题
author: 深 呼吸
type: post
date: 2017-10-15T08:15:42+00:00
url: /2017/10/php-intval-precision/
categories:
  - Develop
  - PHP

---
在 PHP 和其他一些语言都会存在这个问题，转换浮点数为整数的时候会出现精度丢失，如下：

```
$n="19.99"; print intval($n*100); // prints 1998
```

### 解决办法：

1.转换成字符串再 intval

```php
print intval(strval($n*100)); // prints 1999
```

2.使用 round 函数替代 floatval

```php
php -r "echo round(19.99*100);"
```

<!--more-->

3.先转换成字符串再取整

```php
print intval(strval($n*100)); // prints 1999
```

4.使用 bc 系列函数

具体原因：<a href="http://www.laruence.com/2013/03/26/2884.html" target="_blank" rel="noopener nofollow">http://www.laruence.com/2013/03/26/2884.html</a>

