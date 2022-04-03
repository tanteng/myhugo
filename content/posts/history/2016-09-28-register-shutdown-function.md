---
title: PHP捕捉异常中断
author: 深 呼吸
type: post
date: 2016-09-28T09:31:50+00:00
url: /2016/09/register-shutdown-function/
categories:
  - Develop
  - PHP

---
当 PHP 程序出现异常情况，如出现致命错误，超时，或者不可知的逻辑错误导致程序中断，这个时候可以用 register\_shutdown\_function 进行异常处理。

<!--more-->

比如判断一个脚本是否执行完成，可以设置一个属性为 false，在执行完成时设为 true，最后通过 register\_shutdown\_function 函数指定的方法进行判断，并做进一步异常处理，如代码所示：

```php
class IndexController extends Controller
{
    /**
     * 脚本执行是否完成
     * @var bool
     */
    protected $complete = false;

    public function __construct()
    {
        register_shutdown_function([$this, 'shutdown']);
    }

    /**
     * 异常处理
     */
    public function shutdown()
    {
        if ($this-&gt;complete === false) {
            dump('www.tanteng.me'); //此处应该输出日志并进行异常处理操作
        }
    }
}
```

这样一来，可以快速定位脚本是否中断，通过 register\_shutdown\_function 处理异常并提高程序的健壮性，并且可以记录程序中断的状态，方便通过日志快速定位问题。

### register\_shutdown\_function 执行机制

引用：

> PHP 把要调用的函数调入内存。当页面所有 PHP 语句都执行完成时，再调用此函数。注意，在这个时候从内存中调用，不是从 PHP 页面中调用，所以如果有路径信息，应使用绝对路径，因为 PHP 已经当原来的页面不存在了。就没有什么相对路径可言。
> 
> **可以这样理解调用条件：**
  
> 1、当页面被用户强制停止时
  
> 2、当程序代码运行超时时
  
> 3、当ＰＨＰ代码执行完成时

#### register\_shutdown\_function 官方手册 {.refname}

<a href="http://php.net/manual/zh/function.register-shutdown-function.php" target="_blank" rel="nofollow">http://php.net/manual/zh/function.register-shutdown-function.php</a>