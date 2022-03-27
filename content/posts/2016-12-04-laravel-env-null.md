---
title: Laravel 使用 env 读取环境变量为 null 的问题
author: 小谈
type: post
date: 2016-12-04T15:35:18+00:00
url: /2016/12/laravel-env-null/
categories:
  - Develop
  - Laravel
  - PHP

---
不知道大家有没有遇到过，在 Laravel 中（除 app/config 目录下的配置文件中）使用 env 函数读取环境变量，有时有用，有时返回 null，究竟怎么回事？让我们一探究竟。

在 Laravel 项目中，如果执行了 php artisan config:cache 命令把配置文件缓存起来后，在 Tinker 中（Tinker 是 Laravel 自带的一个交互式命令行界面），使用 env 函数读取环境变量的值为 null，只有执行 php artisan config:clear 清除配置缓存后就可以读取了，这是为什么呢？

<!--more-->

打开 .env 文件看，这些都是有值的：

```ini
APP_ENV=local
APP_KEY=base64:JHE5bOkRg283uT0n1Zq/GgvGEer8ooYiB42/wIcCyvo=
APP_DEBUG=true
APP_LOG_LEVEL=debug
APP_URL=http://www.tanteng.me

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=tanteng.me
DB_USERNAME=homestead
DB_PASSWORD=secret
```

### 原因何在？

在 Laravel 中，如果执行 php aritisan config:cache 命令，Laravel 将会把 app/config 目录下的所有配置文件“编译”整合成一个缓存配置文件到  bootstrap/cache/config.php，每个配置文件都可以通过 env 函数读取环境变量，这里是可以读取的。但是一旦有了这个缓存配置文件，在其他地方使用 env 函数是读取不到环境变量的，所以返回 null.

让我们看看这段代码，Illuminate/Foundation/Bootstrap/DetectEnvironment.php line 18：

```php
public function bootstrap(Application $app)
{
    if (! $app-&gt;configurationIsCached()) {
        $this-&gt;checkForSpecificEnvironmentFile($app);

        try {
            (new Dotenv($app-&gt;environmentPath(), $app-&gt;environmentFile()))-&gt;load();
        } catch (InvalidPathException $e) {
            //
        }
    }
}
```

这个方法在框架启动后就会运行，这段代码说明了如果存在缓存配置文件，就不会去设置环境变量了，配置都读缓存配置文件，而不会再读环境变量了。

因此，在配置文件即 app/config 目录下的其他地方，读取配置不要使用 env 函数去读环境变量，这样你一旦执行 php artisan config:cache 之后，env 函数就不起作用了。所有要用到的环境变量，在 app/config 目录的配置文件中通过 env 读取，其他地方要用到环境变量的都统一读配置文件而不是使用 env 函数读取。

这个问题以前遇到过后来改了写法，在 github 上一个扩展包中发现一个 bug，发现也是这个问题导致的，跟作者反馈也确认这一点。