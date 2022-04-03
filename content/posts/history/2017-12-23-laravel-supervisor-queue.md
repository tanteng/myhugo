---
title: Laravel Redis 多个进程同时取队列问题
author: 深 呼吸
type: post
date: 2017-12-23T11:26:19+00:00
url: /2017/12/laravel-supervisor-queue/
categories:
  - Develop
  - Laravel
  - PHP

---
**开启多个进程处理队列会重复读取 Redis 中队列吗？是否因此导致重复执行任务？**

使用 Supervisor 监听 Laravel 队列任务，其中 Supervisor 的配置如下：

<code class="lang:default decode:true">[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/xxx.cn/artisan queue:work --queue=sendfile --tries=3 --daemon
autostart=true
autorestart=true
numprocs=8
redirect_stderr=true
stdout_logfile=/var/www/xxx.cn/worker.log
</code>

注意 numprocs = 8，代表开启 8 个进程来执行 command 中的命令。

<!--more-->

如下：

<code class="lang:default decode:true">PS C:\Users\tanteng\website\laradock&gt; docker-compose exec php-worker sh
/etc/supervisor/conf.d # ps -ef | grep php
    7 root       0:00 php /var/www/xxx.cn/artisan queue:work --queue=sendfile --tries=3 --daemon
    8 root       0:00 php /var/www/xxx.cn/artisan queue:work --queue=sendfile --tries=3 --daemon
    9 root       0:00 php /var/www/xxx.cn/artisan queue:work --queue=sendfile --tries=3 --daemon
   10 root       0:00 php /var/www/xxx.cn/artisan queue:work --queue=sendfile --tries=3 --daemon
   11 root       0:00 php /var/www/xxx.cn/artisan queue:work --queue=sendfile --tries=3 --daemon
   12 root       0:00 php /var/www/xxx.cn/artisan queue:work --queue=sendfile --tries=3 --daemon
   13 root       0:00 php /var/www/xxx.cn/artisan queue:work --queue=sendfile --tries=3 --daemon
   14 root       0:00 php /var/www/xxx.cn/artisan queue:work --queue=sendfile --tries=3 --daemon
   44 root       0:00 grep php</code>

### Laravel 多进程读取队列内容是否会重复

在 Laravel 的某个控制器方法，一次放入多个任务队列：

<code class="lang:default decode:true ">public function index(Request $request)
{
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
    $this-&gt;dispatch((new SendFile3())-&gt;onQueue('sendfile'));
}</code>

在队列处理的方法打印日志，打印处理的队列的 ID：

app/Jobs/SendFile3.php

<code class="lang:default decode:true">public function handle()
{
    info('invoke SendFile3');
    dump('invoke handle');
    $rawbody = $this-&gt;job-&gt;getRawBody();
    $info = json_decode($rawbody, true);
    info('queue id:' . $info['id']);
}</code>

Laravel 使用 Redis 的 list 作为队列的数据结构，并会为每个队列分配一个 ID，数据结构如下：

<code class="lang:js decode:true">{
  "job": "Illuminate\\Queue\\CallQueuedHandler@call",
  "data": {
    "commandName": "App\\Jobs\\SendFile3",
    "command": "O:18:\"App\\Jobs\\SendFile3\":4:{s:6:\"\u0000*\u0000job\";N;s:10:\"connection\";N;s:5:\"queue\";s:8:\"sendfile\";s:5:\"delay\";N;}"
  },
  "id": "hadBcy3IpNsnOofQQdHohsa451OkQs88",
  "attempts": 1
}</code>

请求这个控制器路由（或者命令行方式），就可以看到 Redis 中多了很多队列任务了，如图：

<img class="alignnone wp-image-11875 size-full" src="https://blog.tanteng.me/wp-content/uploads/2017/12/redis_laravel_20171223191227.png" alt="" width="1195" height="619" />

这个时候开启 Supervisor 处理队列任务，并查看日志：

<code class="lang:default decode:true">[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:JaClJzhDEvntzLCRIz6uRQkCVLbE8Y9C
[2017-12-23 19:01:01] local.INFO: queue id:ukHv0Li4P2VgPa55qU6yEOJM27Mo5YwJ
[2017-12-23 19:01:01] local.INFO: queue id:ObMpwDTmnaveBUkU7aan5abt3Agyt90l
[2017-12-23 19:01:01] local.INFO: queue id:fo2qZn2ftSdQtdnKOciMK7iJb4qlhRGE
[2017-12-23 19:01:01] local.INFO: queue id:uLjFMoOU7Wk7bOAd4zpHb3ccRMJHBtR6
[2017-12-23 19:01:01] local.INFO: queue id:87ULqPBObFmGr16nl5wxFVOi71zGCeRM
[2017-12-23 19:01:01] local.INFO: queue id:9UVl0muQLzBqlRI99rChGW2ElXwVEMIE
[2017-12-23 19:01:01] local.INFO: queue id:a0vgyZuz9HtmH7DGHEpXqesFTcQU3QAF
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:2cXuXxopPkgYiV4WO8gv9CJ6CwXeKtYL
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:9acTAYa8cxpJX6Q3Gb1sULokotP8reqZ
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:BPHQvBboChlv4gr2I0vyLVyw9bijtTYJ
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:Fm6tNajdxYKtdQbDMYDmwWJFLnNikRyg
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:nyAbcvSkBVPbaH3e2ItQkoLJlP1ficib
[2017-12-23 19:01:01] local.INFO: queue id:WBHsSVZtP43569UoPXxfLLJcvYmPW7cP
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:bliPnKcRSDApwVmKLNxEhaKelhm0RDEY
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:eOAoQucEIwRz9uZ64xm6IDKgiqj9Xc3W
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:lzise9EiqQqINrhALbmAI4qNg7qylpb2
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:WXYKvcfOhS1pPnwOwUTsenoMv5l5EUXe
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:XtH5JiwLgnrwWzI02Oyi70pihAOkuJUD
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:9ehmE5HImlpNubpY0xWN8UVrOzxeMqws
[2017-12-23 19:01:01] local.INFO: queue id:C1sK87cpZl47edLA0zhfo7PJ9MIEcoyx
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:2kwl51oH4lyyRrljCReGUCkNiJRDl7oe
[2017-12-23 19:01:01] local.INFO: queue id:ObRpoqrYTPYiyv2delMlOXu3sAPpWJlN
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:6qgu6W3TapLjSrt688yv9HRXvDDLxntz
[2017-12-23 19:01:01] local.INFO: queue id:wiTlERhwn7s9cQkfUF9lLlNADpXjKncI
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:ZSLW0VLFBDpL4wjTJzu3Yb3V45pNe807
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:qhZlXLGfGWRluIeNm7VbllmTJZYb2h5n
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:LUx1IByD3L2psNl9BZwHhk2knXyRPzW6
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:M2RESPjyo5hpAFxxL0EQbWwsUq4jpmWn
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:hUsGaiIAOO6ZfGQc5kGHGpsv5RpoRPYO
[2017-12-23 19:01:01] local.INFO: queue id:cEHJsOy6bLeZ4NbncPziaHqlarMeyyEF
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:w4bkFiJKMU5saqG2xKN3ZRL5BYXGATMk
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:0zBuwbxlrEhhxKfYBkVyTY4z35f154sI
[2017-12-23 19:01:01] local.INFO: queue id:mvoZvyDPvq4tcPjEy9G7PMtH3MwPkPik
[2017-12-23 19:01:01] local.INFO: invoke SendFile3
[2017-12-23 19:01:01] local.INFO: queue id:TLvF74eeidECWKtjZqWvW03UJTRPTL9r
[2017-12-23 19:01:01] local.INFO: queue id:me8wyPfgcz0nf9xvcXz0hf2xVxqa1FFS</code>

这 8 个进程并发处理队列，但从打印的日志看，没有出现同样的 ID. 我们再看一下 Laravel 如何使用 Redis 处理队列的。

### 分析一下 Laravel 队列的处理

#### Laravel 中入队列方法

<code class="lang:php decode:true ">public function pushRaw($payload, $queue = null, array $options = [])
{
    $this-&gt;getConnection()-&gt;rpush($this-&gt;getQueue($queue), $payload);

    return Arr::get(json_decode($payload, true), 'id');
}</code>

用的是 Redis 的 rpush 命令。

#### Laravel 中取队列方法

<code class="lang:php decode:true ">public function pop($queue = null)
{
    $original = $queue ?: $this-&gt;default;

    $queue = $this-&gt;getQueue($queue);

    $this-&gt;migrateExpiredJobs($queue.':delayed', $queue);

    if (! is_null($this-&gt;expire)) {
        $this-&gt;migrateExpiredJobs($queue.':reserved', $queue);
    }

    list($job, $reserved) = $this-&gt;getConnection()-&gt;eval(
        LuaScripts::pop(), 2, $queue, $queue.':reserved', $this-&gt;getTime() + $this-&gt;expire
    );

    if ($reserved) {
        return new RedisJob($this-&gt;container, $this, $job, $reserved, $original);
    }
}</code>

这里用的是 lua 脚本取队列，如下：

<code>public static function pop()
{
    return &lt;&lt;&lt;'LUA'
local job = redis.call('lpop', KEYS[1])
local reserved = false
if(job ~= false) then
reserved = cjson.decode(job)
reserved['attempts'] = reserved['attempts'] + 1
reserved = cjson.encode(reserved)
redis.call('zadd', KEYS[2], ARGV[1], reserved)
end
return {job, reserved}
LUA;
}</code>

那么结论是：**从 Laravel 的处理方式和打印的日志结果看，即使多个进程读取同一个队列，也不会读取到一样的数据。**

#### 参考文章：

<a href="https://www.lijinma.com/blog/2017/01/31/laravel-queue/" target="_blank" rel="noopener nofollow">关于 Laravel 入队列和消费队列的具体过程可以看这篇文章，讲得非常详细</a>