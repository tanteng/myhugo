---
title: Laravel 5.3 后台管理系统搭建
author: 小谈
type: post
date: 2016-12-24T07:13:14+00:00
url: /2016/12/laravel-admin-install/
duoshuo_thread_id:
  - 6367562160075178753
categories:
  - Develop
  - Laravel
  - PHP

---
网上的很多基于 Laravel 的后台管理系统，要么缺少基本的功能，要么太臃肿，总是找不到自己喜欢的，于是自己做了一个。它的特点是尽可能简单又不缺少基本的后台功能，如用户认证，基于角色的权限系统（Entrust），菜单管理，媒体管理等，并且使用 AdminLte 后台界面，外观简洁功能强大。

<!--more-->

下面讲一讲 Laravel-admin 的安装步骤，或者一些不要的说明，可以使用本系统，但是更建议参考本后台搭建的方法自行弄一套更符合自己的后台管理系统，这种后台管理系统是可以复用到不同的项目中的。

### 一、安装扩展包

使用自带的用户认证，权限管理基于 Entrust 扩展包，实现基于角色的权限控制系统，用于开发调试的 laravel-debugbar 扩展包，还有必不可少的操作 Redis 的扩展包。

本后台依赖的扩展包：

Entrust

Predis

laravel-debugbar

在 composer.json 中添加扩展包：

<code class="lang:default decode:true">"predis/predis": "^1.1",
"zizaco/entrust": "5.2.x-dev",
"barryvdh/laravel-debugbar": "^2.3"</code>

推荐安装最新版本。执行 sudo composer install 即可。

**重要：具体安装好扩展包之后，如果还需要进行的步骤，参考扩展包的官方 github 主页。**

这里自行还要操作的步骤还很多，如 Entrust 需要执行数据库迁徙文件，建立 Role 和 Permission 的模型，并且使用 Traits 等，按照 Entrust 的安装说明进行。关于 Entrust ，注意修改 config/entrust.php 中的模型对应的实际的类，如：

<code>'role' =&gt; 'App\Models\Role',</code>

#### 扩展包 github 主页：

Entrust：<a href="https://github.com/Zizaco/entrust" target="_blank" rel="nofollow">https://github.com/Zizaco/entrust</a>

Laravel-debugbar：<a href="https://github.com/barryvdh/laravel-debugbar" target="_blank" rel="nofollow">https://github.com/barryvdh/laravel-debugbar</a>

Predis: <a href="https://github.com/nrk/predis" target="_blank" rel="nofollow">https://github.com/nrk/predis</a>

### 二、安装 AdminLte 后台模板

AdminLte 是一套完整的专业的后台模板，由静态页面和很多 js 组件构建，UI 简洁美观，github 上 Stars 数量很多，可以用来做通用的后台管理系统界面。

github 地址：https://github.com/almasaeed2010/AdminLTE

这个就比较麻烦，把需要用到的 css，js，插件等文件夹复制到项目的 pubic 文件夹下，可以新建一个目录如 vendor，然后把登陆页面，后台主页面等 html 做成 blade 模板放到模板目录，如 resource/views/admin 目录下。

### 三、后台管理路由

以 Laravel 5.3 为例，在 routes 目录下新增 admin.php 文件，在这个文件定义后台管理的路由，基本路由示例如下：

<code>&lt;?php
Route::get('/', function () {
    return view('admin.index');
})-&gt;middleware('auth:admin');
Route::get('/login', 'Admin\LoginController@showLoginForm');
Route::post('/login', 'Admin\LoginController@login');
Route::get('/logout', 'Admin\LoginController@logout');
Route::resource('/user', 'Admin\UserController');
Route::resource('/role', 'Admin\RoleController');
Route::resource('/permission', 'Admin\PermissionController');
Route::get('testEntrust', 'Admin\TestEntrustController@hello');
</code>

在 app/Providers/RouteServiceProvider.php 中添加方法，使用 web 中间件，并设置路由前缀为 admin，具体如下：

<code>/**
 * 后台管理路由
 */
protected function mapAdminRoutes()
{
    Route::group([
        'middleware' =&gt; 'web',
        'namespace' =&gt; $this-&gt;namespace,
        'prefix' =&gt; 'admin',
    ], function ($router) {
        require base_path('routes/admin.php');
    });
}</code>

别忘了在 map 方法中调用新增的方法，这样一个后台管理基本界面就好了。

### 四、后台 Auth 认证

#### 增加一个 guard {.}

<p class="">
  后台身份认证和前台可以实现分开，在 config/auth.php 的 guards 数组中添加后台的 guard 定义：
</p>

<code class="lang:php decode:true ">'guards' =&gt; [
    'web' =&gt; [
        'driver' =&gt; 'session',
        'provider' =&gt; 'users',
    ],

    'admin' =&gt; [
        'driver' =&gt; 'session',
        'provider' =&gt; 'users',
    ],

    'api' =&gt; [
        'driver' =&gt; 'token',
        'provider' =&gt; 'users',
    ],
],</code>

这里新增的 admin 定义跟 web 一样，只是共用 users 表作为后台的用户表，这里严格说应该分开，甚至把后台的数据库也独立，本文作为一个示例就不做很完善，这样也会把过程弄得很复杂，大概的思路 OK 了，可以根据自己的情况去搭建后台。

#### 判断后台未登陆跳转后台登陆页面

<p class="">
  Laravel 5.3 将未登录作为一个异常进行处理，如果是后台未登录，则跳转到后台的登陆页面，需要在 app/Exceptions/Handler.php 文件修改 unauthenticated 方法如下：
</p>

<code>protected function unauthenticated($request, AuthenticationException $exception)
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
}</code>

#### 修改中间件

如果已经登录后台，进入后台登录页面跳转到后台首页而无需再次登录，这个是由一个名为 guest 的中间件控制的。而默认会跳转到前台的页面，需要进行修改，文件位于 app/Http/Middleware/RedirectIfAuthenticated.php ，其中修改如下：

<code class="lang:php decode:true ">public function handle($request, Closure $next, $guard = null)
{
    if (Auth::guard($guard)-&gt;check()) {
        $redirect = $guard == 'admin' ? '/admin' : '/home';
        return redirect($redirect);
    }

    return $next($request);
}</code>

这样已经登录后台之后，再进入后台登录页面，会跳转到后台首页了，而不是前台页面。

### 五、后台管理控制器

后台管理控制器都在目录 app/Http/Controllers/Admin 下，定义好了需要用到的模型以及模型的关系后，那么主要的逻辑在这些控制器中，这是一些基本的后台功能。

至此，一个全新的基于 Laravel 5.3 的后台管理系统就搭建好了，有了这个后台，可以开发强大的功能了。