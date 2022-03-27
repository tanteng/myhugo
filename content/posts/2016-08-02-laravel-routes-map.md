---
title: Laravel 分割 routes.php 路由文件的一种方式
author: 深 呼吸
type: post
date: 2016-08-02T09:20:26+00:00
url: /2016/08/laravel-routes-map/
categories:
  - Develop
  - Laravel
  - PHP

---
Laravel 的路由功能很强大，路由规则默认都定义在 routes.php 文件中，但是随着项目越来越大，我们需要的定义的规则越来越多，如果几百上千个路由都定义在一个文件中，如何去维护？如果不同的人都在同一个文件定义路由，这就造成了冲突，因此我们有必要将 routes.php 文件分割成多个文件，可以按照功能模块来划分，下面介绍一种很优雅的方式。

在 app/Providers/RouteServiceProvider.php 的 map 方法中可以如下定义：

```php
public function map(Router $router)
{
    $router->group(['namespace' => $this->namespace], function ($router) {
        //require app_path('Http/routes.php');
        foreach (glob(app_path('Http//Routes') . '/*.php') as $file) {
            $this->app->make('App\\Http\\Routes\\' . basename($file, '.php'))->map($router);
        }
    });
}
```

这样它就会遍历 app/Http/Routes/ 文件夹下的文件，遍历每个文件的 map 方法，其中每个文件的结构都类似，举个例子：

```php
<?php
namespace App\Http\Routes;

use Illuminate\Contracts\Routing\Registrar;

class HomeRoutes
{
    public function map(Registrar $router)
    {
        $router->group(['domain' => 'www.tanteng.me', 'middleware' => 'web'], function ($router) {
            $router->auth();
            $router->get('/', ['as' => 'home', 'uses' => 'IndexController@index']);
            $router->get('/blog', ['as' => 'index.blog', 'uses' => 'BlogController@index']);
            $router->get('/resume', ['as' => 'index.resume', 'uses' => 'IndexController@resume']);
            $router->get('/post', ['name' => 'post.show', 'uses' => 'ArticleController@show']);
            $router->get('/contact', ['as' => 'index.contact', 'uses' => 'IndexController@contact']);
            $router->post('/contact/comment', ['uses' => 'IndexController@postComment']);
            $router->get('/travel', ['as' => 'index.travel', 'uses' => 'TravelController@index']);
            $router->get('/travel/latest', ['as' => 'travel.latest', 'uses' => 'TravelController@latest']);
            $router->get('/travel/{destination}/list', ['as' => 'travel.destination', 'uses' => 'TravelController@travelList']);
            $router->get('/travel/{slug}', ['uses' => 'TravelController@travelDetail']);
            $router->get('/sitemap.xml', ['as' => 'index.sitemap', 'uses' => 'IndexController@sitemap']);
        });
    }
}
```

通过把路由规则分割写到不同的文件中，这样一来，就可以根据功能模块分开管理路由文件了。此外，你也可以简单的分割，直接把 routes.php 中的定义拆散成多个文件，通过 require 的方式引入，但是哪个更好，一目了然。

那么这样路由分开多个文件后岂不是增加调用次数，会不会影响性能？答案是不必担心。通过 Laravel 的命令：

```php
php artisan route:cache
```

生成路由缓存文件后，路由只会读取缓存文件的路由规则，因此不会影响性能，这样做让开发更高效和规范。