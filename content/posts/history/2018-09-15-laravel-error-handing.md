---
title: Laravel 错误和异常处理用法
author: 深 呼吸
type: post
date: 2018-09-15T05:41:51+00:00
url: /2018/09/laravel-error-handing/
description: Laravel 自带错误和异常处理，App\Exceptions\Handler 负责上报异常和如何返回内容，以及未登录的处理。App\Exceptions\Handler 位于 app\Exceptions\Handler.php，下面介绍这个类的属性和用法。
categories:
  - Develop
  - Laravel
  - PHP

---
Laravel 自带错误和异常处理，App\Exceptions\Handler 负责上报异常和如何返回内容，以及未登录的处理。App\Exceptions\Handler 位于 app\Exceptions\Handler.php，下面介绍这个类的属性和用法。
  
<!--more-->

### 忽略异常

在 $dontReport 中可以定义忽略的异常类名：

```
protected $dontReport = [
    \Illuminate\Auth\AuthenticationException::class,
    \Illuminate\Auth\Access\AuthorizationException::class,
    \Symfony\Component\HttpKernel\Exception\HttpException::class,
    \Illuminate\Database\Eloquent\ModelNotFoundException::class,
    \Illuminate\Validation\ValidationException::class,
];
```

这些异常就不会经过 report 方法。

### 几个重要方法

主要介绍这三个方法，report，render 和 unauthenticated 的用法。

#### eport方法

<p data-source-line="24">
  report 方法可以用来记录日志，可以根据不同的异常类型（包括自定义异常类型），如 ClientException，ConnectException 定制不同的日志级别和日志内容。
</p>

```php
if ($exception instanceof ABCException) {
    Log::emergency('ABC异常', $context);
} else if ($exception instanceof HeheException) {
    Log::info('Hehe异常', $context);
}
```

<p data-source-line="34">
  report 方法没有返回值，也不应该在这里中断程序。
</p>

<h4 id="render方法" data-source-line="36">
  render方法
</h4>

<p data-source-line="38">
  render 方法可以根据不同的异常类型，返回不同的数据。如：
</p>

```php
if (get_class($exception) == 'Exception' || $exception instanceof NotAllowedException) {
    return response()-&gt;json(['message' =&gt; $exception-&gt;getMessage()], 400);
} elseif ( $exception instanceof ValidationException) {
    return response()-&gt;json(['message' =&gt; '校验失败', 'errors'=&gt; $exception-&gt;validator-&gt;errors()], 400);
}
```

<h4 id="unauthenticated" data-source-line="48">
  unauthenticated
</h4>

<p data-source-line="50">
  在访问需要登录态的页面时，用户未登录就会进入这个方法进行处理，举个例子说明：
</p>

```php
protected function unauthenticated($request, AuthenticationException $exception)
{
    if ($request-&gt;expectsJson()) {
        return response()-&gt;json(['error' =&gt; 'Unauthenticated.'], 401);
    }

    //如果是后台页面未认证,跳转到后台登陆页面
    $guard = $exception-&gt;guards();
    if (in_array('admin', $guard)) {
        return redirect()-&gt;guest('/admin/login');
    }

    return redirect()-&gt;guest('login');
}
```

<p data-source-line="69">
  如果是返回 json，则统一返回格式。
</p>

<p data-source-line="71">
  默认情况下返回前台的登录页，如果是访问后台页面未登录，则跳转到后台登录页。
</p>

<h4 id="官方文档" data-source-line="48">
  官方文档
</h4>

<p data-source-line="50">
  Laravel 5.6
</p>

<p data-source-line="52">
  <a href="https://laravel-china.org/docs/laravel/5.6/errors/1373" target="_blank" rel="noopener nofollow">https://laravel-china.org/docs/laravel/5.6/errors/1373</a>
</p>