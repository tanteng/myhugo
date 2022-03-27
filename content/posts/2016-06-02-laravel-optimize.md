---
title: 优化 Laravel 网站打开速度
author: 小谈
type: post
date: 2016-06-02T11:59:07+00:00
url: /2016/06/laravel-optimize/
categories:
  - Develop
  - Laravel
  - PHP

---
Laravel是一个功能强大的框架，组件很多，代码也很庞大，它的易用方便是牺牲了性能的，即便如此它仍然是一个优秀的框架，但在正式环境下要做好优化提升网站的打开速度。

### 1.关闭debug

打开.env文件，把debug设置为false.

```ini
APP_ENV=local
APP_DEBUG=false
APP_KEY=base64:sT/aTFeaE13eyao1Raee6jC9Ff+Yle1SE+wtyk0H6B4=
```

### 2.缓存路由和配置

php artisan route:cache

php artisan config:cache


### 3.Laravel优化命令

php artisan optimize

### 4.composer优化

sudo composer dump-autoload optimize

### 5.使用Laravel缓存

使用Laravel的Cache方法缓存内容，有文件缓存，数据库缓存，redis缓存，使用redis也可以用predis组件，也可以多种缓存方式结合。在Laravel中使用缓存就是这么优雅方便：

```php
$lists = Cache::remember('travel.destination.lists', 20, function () {
    return $this-&gt;destination-&gt;getList();
});
```

### 6.使用CDN

本站用的是七牛CDN，每月送你20G流量和20G存储空间，具体多少不记得了，总之对于小站来说完全足够了。

### 7.使用PHP 7并开启OPcache

### 8.nginx开启gzip压缩

这不仅仅是针对Laravel网站的性能优化方法，其中很多是通用的网站性能优化的方法，当然还有很多可以优化的地方。

可以把以上提到的优化命令写成一个脚本：

```
#!/usr/bin/env bash
php artisan clear-compiled
php artisan cache:clear
php artisan route:cache
php artisan config:cache
php artisan optimize --force
composer dump-autoload --optimize
chmod -R 777 storage
chmod -R 777 bootstrap/cache
```

命名为 optimize.sh 放网站根目录，这样可以方便执行。
