---
title: 初识PHP Static延迟静态绑定
author: 深 呼吸
type: post
date: 2015-05-27T03:38:42+00:00
url: /2015/05/php-static-bind/
duoshuo_thread_id:
  - 1.2310560994115E+18
categories:
  - Develop
  - PHP

---
PHP5.3以后引入了延迟静态绑定static，它是为了解决什么问题呢？php的继承模型中有一个存在已久的问题，那就是在父类中引用扩展类的最终状态比较困难。来看一个例子。

<code>class A  
{  
    public static function echoClass(){  
        echo __CLASS__;  
    }
  
    public static function test(){  
        self::echoClass();        
    }
}
  
class B extends A  
{        
    public static function echoClass()  
    {  
         echo __CLASS__;  
    }  
}  
  
B::test(); //输出A</code>

<!--more-->

在PHP5.3中加入了一个新特性：延迟静态绑定，就是把本来在定义阶段固定下来的表达式或变量，改在执行阶段才决定，比如当一个子类继承了父类的静态表达式的时候,它的值并不能被改变，有时不希望看到这种情况。

下面的例子解决了上面提出的问题：

<code class="lang:php mark:9 decode:true ">class A  
{  
    public static function echoClass(){  
        echo __CLASS__;  
    }  
  
    public static function test()  
    {  
        static::echoClass();        
    }  
}  
  
class B extends A  
{        
    public static function echoClass(){  
         echo __CLASS__;  
    }  
}  
  
B::test(); //输出B</code>

第9行定义了一个静态延迟绑定方法，直到B调用test的时候才执行原本定义的时候执行的方法。

转载自：http://blog.csdn.net/suiye/article/details/8729511