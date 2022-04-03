---
title: 为什么 Laravel 会重复执行同一个队列任务？
author: 小谈
type: post
date: 2017-12-24T07:49:30+00:00
url: /2017/12/laravel-queue-excute/
categories:
  - Develop
  - PHP

---
在 Laravel 中使用 Redis 处理队列任务，框架提供的功能非常强大，但是最近遇到一个问题，就是发现一个任务被多次执行，这是为什么呢？

先说原因：因为在 Laravel 中如果一个队列（任务）执行时间大于 60 秒，就会被认为执行失败并重新加入队列中，这样就会导致重复执行同一个任务。

备注一下：这里的 Laravel 版本是 5.3，据说后面的版本修复了这个问题。

这个任务的逻辑就是给用户推送内容，需要根据队列内容取出用户并遍历，通过请求后端 HTTP 接口发送。比如有 10000 个用户，在用户数量多或接口处理速度没那么快的情况下，执行时间肯定会大于 60 秒，于是这个任务就被重新加入队列。**情况更糟糕一点，前面的任务如果都没有在 60 秒执行完，就都会重新加入队列，这样同一个任务就不止重复执行一次了，而是多次。**

下面从 Laravel 源代码找一下罪魁祸首。

源代码文件：vendor/laravel/framework/src/Illuminate/Queue/RedisQueue.php

```php
/**
 * The expiration time of a job.
 *
 * @var int|null
 */
protected $expire = 60;
```

这个 $expire 成员变量是一个固定的值，Laravel 认为一个队列再怎么 60 秒也该执行完了吧。取队列方法：

```php
public function pop($queue = null)
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
}
```

取队列有几步操作，因为队列执行失败，或执行超时等都会放入另外的集合保存起来，以便重试，过程如下：

1.把因执行失败的队列从 delayed 集合重新 rpush 到当前执行的队列中。

2.把因执行超时的队列从 reserved 集合重新 rpush 到当前执行的队列中。

3.然后才是从队列中取任务开始执行，同时把队列放入 reserved 的有序集合。

这里使用了 eval 命令执行这个过程，用到了几个 lua 脚本。

从要执行的队列中取任务：

```php
local job = redis.call('lpop', KEYS[1])
local reserved = false
if(job ~= false) then
    reserved = cjson.decode(job)
    reserved['attempts'] = reserved['attempts'] + 1
    reserved = cjson.encode(reserved)
    redis.call('zadd', KEYS[2], ARGV[1], reserved)
end
return {job, reserved}
```

**可以看到 Laravel 在取 Redis 要执行的队列的时候，同时会放一份到一个有序集合中，并使用过期时间戳作为分值。**

只有当这个任务完成后，再把有序集合中这个任务移除。从这个有序集合移除队列的代码就省略，**我们看一下 Laravel 如何处理执行时间大于 60 秒的队列**。

也就是这段 lua 脚本执行的操作：

```php
local val = redis.call('zrangebyscore', KEYS[1], '-inf', ARGV[1])
if(next(val) ~= nil) then
    redis.call('zremrangebyrank', KEYS[1], 0, #val - 1)
    for i = 1, #val, 100 do
        redis.call('rpush', KEYS[2], unpack(val, i, math.min(i+99, #val)))
    end
end
return true
```

这里 zrangebyscore 找出分值从无限小到当前时间戳的元素，也就是 60 秒之前加入到集合的任务，然后通过 zremrangebyrank 从集合移除这些元素并 rpush 到队列中。

看到这里应该就恍然大悟了。

如果一个队列 60 秒没执行完，那么进程在取队列的时候从 reserved 集合中把这些任务又重新 rpush 到队列中。
