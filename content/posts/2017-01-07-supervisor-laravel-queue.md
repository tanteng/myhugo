---
title: 使用 Supervisor 管理 Laravel 队列进程
author: 小谈
type: post
date: 2017-01-07T12:48:53+00:00
url: /2017/01/supervisor-laravel-queue/
duoshuo_thread_id:
  - 6372843847327679234
categories:
  - Develop
  - Laravel
  - PHP
  - 服务器

---
Supervisor 是一个 Python 写的进程管理工具，有时一个进程需要在后台运行，并且意外挂掉后能够自动重启，就需要这么一个管理进程的工具。在 Laravel 开发中，也经常使用到队列监听，可以配合 Supervisor 来管理 Laravel 队列进程。

<!--more-->

### Supervisor的安装

1. 使用 pip 工具进行安装：

<code class="lang:python decode:true ">sudo pip install supervisor</code>

2. Ubuntu 系统使用 apt-get

<code class="lang:default decode:true ">sudo apt-get install supervisor</code>

还有其他的安装方式，请见官网（<a href="http://supervisord.org/" target="_blank" rel="nofollow">http://supervisord.org/</a>）

### Supervisor的配置

运行这个命令可以生成一个默认的配置文件：

<code class="lang:default decode:true ">echo_supervisord_conf &gt; /etc/supervisord.conf</code>

生成成功后，打开编辑这个文件，把最后的 include 块的注释打开，并修改如下：

<code class="lang:default decode:true">[include]
files = /etc/supervisor/*.conf</code>

新增的 Supervisor 配置文件放在 /etc/supervisor 目录下，并且以 conf 结尾。

这时我们使用新的配置文件来启动 Supervisor：

<code class="lang:default decode:true ">supervisord -c /etc/supervisord.conf</code>

如果提示已经有进程在运行，那么先 kill 掉它。

### 使用Supervisor管理Laravel队列进程

我们使用 Laravel 队列，会用到 php artisan queue:work 命令，让它监听队列，我们可以通过 nohup 方式让它在后台运行，但是进程如果意外中断是不会自动重启的，所以使用 Supervisor 来监控进程是个很好的方式。

首先在 /etc/supervisor 目录下新增一个 Supervisor 的配置文件，如下：

<code class="lang:default decode:true">[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /usr/share/nginx/html/tanteng.me/artisan queue:work --tries=3
autostart=true
autorestart=true
user=vagrant
numprocs=8
redirect_stderr=true
stdout_logfile=/var/log/supervisor/laravel-queue.log</code>

这里 user 填写网站运行进程的用户，如 vagrant，numprocs 表示启动多少个进程来监听 Laravel 队列。

一切就绪后，我们使用如下命令就可以启动队列进程的监听了：

<code class="lang:default decode:true ">sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start laravel-worker:*</code>

但是在这一步，发生了错误，提示如下：

laravel-worker:laravel-worker_00: ERROR (spawn error)
  
laravel-worker:laravel-worker_01: ERROR (spawn error)
  
laravel-worker:laravel-worker_02: ERROR (spawn error)
  
laravel-worker:laravel-worker_03: ERROR (spawn error)
  
laravel-worker:laravel-worker_04: ERROR (spawn error)
  
laravel-worker:laravel-worker_05: ERROR (spawn error)
  
laravel-worker:laravel-worker_06: ERROR (spawn error)
  
laravel-worker:laravel-worker_07: ERROR (spawn error)

查看 Supervisor 错误日志，提示如下：

INFO spawnerr: unknown error making dispatchers for &#8216;laravel-worker_03&#8217;: EACCES

原因是权限问题，解决方法是，把 Supervisor 的日志文件，和新增的配置文件中的日志文件，设置正确的用户和组，如本例通过 chown vagrant:vagrant file_name 设置日志文件用户和组权限，另外把日志文件权限设置为 777.

再次经过上述步骤，成功开启进程管理：

laravel-worker:laravel-worker_00: started
  
laravel-worker:laravel-worker_01: started
  
laravel-worker:laravel-worker_02: started
  
laravel-worker:laravel-worker_03: started
  
laravel-worker:laravel-worker_04: started
  
laravel-worker:laravel-worker_05: started
  
laravel-worker:laravel-worker_06: started
  
laravel-worker:laravel-worker_07: started

可以看到 Laravel 队列开始正常运行了，这里值得注意的是，如果 Laravel 处理队列的代码更改了，需要重启 Supervisor 的队列管理才能生效。

### 使用 supervisorctl {#使用-supervisorctl}

Supervisorctl 是 supervisord 的一个命令行客户端工具，启动时需要指定与 supervisord 使用同一份配置文件，否则与 supervisord 一样按照顺序查找配置文件。

<code class="lang:default decode:true ">supervisorctl -c /etc/supervisord.conf</code>

上面这个命令会进入 supervisorctl 的 shell 界面，然后可以执行不同的命令了：

<code class="lang:default decode:true ">&gt; status    # 查看程序状态
&gt; stop usercenter   # 关闭 usercenter 程序
&gt; start usercenter  # 启动 usercenter 程序
&gt; restart usercenter    # 重启 usercenter 程序
&gt; reread    ＃ 读取有更新（增加）的配置文件，不会启动新添加的程序
&gt; update    ＃ 重启配置文件修改过的程序</code>

上面这些命令都有相应的输出，除了进入 supervisorctl 的 shell 界面，也可以直接在 bash 终端运行：

<code class="lang:default decode:true ">$ supervisorctl status
$ supervisorctl stop usercenter
$ supervisorctl start usercenter
$ supervisorctl restart usercenter
$ supervisorctl reread
$ supervisorctl update</code>

更多用法，请参考如下链接。

### 链接

  * <a href="http://liyangliang.me/posts/2015/06/using-supervisor/" target="_blank" rel="nofollow">使用 Supervisor 管理进程</a>
  * <a href="https://laravel.com/docs/5.3/queues#supervisor-configuration" target="_blank" rel="nofollow">Laravel 官方文档之队列管理</a>

&nbsp;