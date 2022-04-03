---
title: 使用 Laravel 消息队列要注意的问题
author: 小谈
type: post
date: 2017-12-11T10:08:13+00:00
url: /2017/12/laravel-queue-tips/
categories:
  - Develop
  - Laravel
  - PHP

---
使用 Laravel 的消息队列处理异步任务，Redis 作为队列数据库，Supervisor 监控脚本异常中断并自动重启，这是 Laravel 处理队列任务的标准流程，但是实际中可能还会出现各种各样的问题，为了保证系统**可靠性**，还要注意几个问题。

### 一、执行失败重试次数设置

一定要设置任务执行失败重试次数，避免无限失败重试，超过重试次数 Laravel 会默认写到失败任务表中，也可以自己写执行失败后续处理逻辑。

```php artisan queue:work redis --tries=3```


需要先执行以下命令创建数据表：

```
php artisan queue:failed-table

php artisan migrate
```

### 二、程序异常的处理

有时候程序执行过程会发生异常，比如依赖其他接口，请求 HTTP 接口超时等等，如果不捕捉异常，那么当前这个队列就会中断不能继续运行下去，比如给 10000 个用户推送内容，需要依赖接口推送，如果中间的请求挂了就会影响到后面的推送。

这里的异常是指程序执行过程中发生的异常，不是指常驻进程挂掉，程序异常不一定导致常驻进程中断，况且进程中断有 Supervisor 监控并重启。

如捕获异常代码片段：

```php
try {
    $r = $client->request('POST', '', [
        'query' => [
            'client_name'     => 'filemail',
            'client_version'  => '1.0',
            'client_sequence' => 0,
            'uid'             => 692934013,//119481237
            'r'               => 1508312484,
        ],
        'body'  => \GuzzleHttp\json_encode($body),
    ]);
    $result = $r->getBody()->getContents();
    $result = json_decode($result, true);
    if ($result['result'] == 0) {
        info("sendMail fail:" . json_encode($result));
        $this->pushLog($task['id'], $task['mail_id'], implode(',', $userIds), json_encode($result), 0);
    } else {
        Log::warning("sendMail fail:" . json_encode($result));
        $this->pushLog($task['id'], $task['mail_id'], implode(',', $userIds), json_encode($result), $result['result']);
    }
} catch (RequestException $e) {
    Log::warning('RequestException' . $e->getMessage());
} catch (Exception $e) {
    Log::emergency('Exception' . $e->getMessage());
}
```

### 三、修改代码记得重启 Supervisor

最后一点，修改了处理队列的程序，记得要重启 Supervisor，否则脚本不会生效。

### Laravel 往 Redis 写队列的数据结构

队列用 list 类型存储.

value 内容如下：

```php
{
  "job": "Illuminate\\Queue\\CallQueuedHandler@call",
  "data": {
    "commandName": "App\\Jobs\\SendFile",
    "command": "O:17:\"App\\Jobs\\SendFile\":5:{s:23:\"\u0000App\\Jobs\\SendFile\u0000task\";a:8:{s:5:\"title\";s:4:\"1111\";s:4:\"note\";s:2:\"11\";s:6:\"reward\";s:0:\"\";s:7:\"mail_id\";s:5:\"66681\";s:4:\"nums\";i:20;s:8:\"uid_file\";s:33:\"uidfile\/file-66681-1513058185.txt\";s:5:\"gcids\";s:40:\"1B9DD95645AAE8119F7DA9B9FF738D52BC8A1BD5\";s:2:\"id\";i:29;}s:6:\"\u0000*\u0000job\";N;s:10:\"connection\";N;s:5:\"queue\";s:8:\"sendfile\";s:5:\"delay\";N;}"
  },
  "id": "l0mjsUthbxm4TgIJNUH13km9N8DIpErK",
  "attempts": 1
}
```

包含失败重试次数，队列标识，处理队列的类，以及队列的数据等等。

### 参考链接

Laravel 官方文档 Queue 队列：

<a href="https://laravel.com/docs/5.5/queues" target="_blank" rel="noopener nofollow">https://laravel.com/docs/5.5/queues</a>