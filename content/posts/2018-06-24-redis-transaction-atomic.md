---
title: Redis 的事务到底是不是原子性的
author: 深 呼吸
type: post
date: 2018-06-24T02:43:46+00:00
url: /2018/06/redis-transaction-atomic/
categories:
  - Redis
  - 事物

---
ACID 中关于原子性的定义：

> 原子性：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被恢复（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。

那么 Redis 的事务到底符不符合原子性的特征呢？官方文档对事务的描述如下：

> 事务可以一次执行多个命令， 并且带有以下两个重要的保证：
> 
> 
>   <ul>
>     <li>
>       事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
>     </li>
>     <li>
>       <strong>事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执行。</strong>EXEC 命令负责触发并执行事务中的所有命令： <ul>
>         <li>
>           如果客户端在使用 MULTI 开启了一个事务之后，却因为断线而没有成功执行 EXEC ，那么事务中的所有命令都不会被执行。
>         </li>
>         <li>
>           另一方面，如果客户端成功在开启事务之后执行 EXEC ，那么事务中的所有命令都会被执行。
>         </li>
>       </ul>
>     </li>
>   </ul>
> 
> 
> 当使用 AOF 方式做持久化的时候， Redis 会使用单个 write(2) 命令将事务写入到磁盘中。
> 
> 然而，如果 Redis 服务器因为某些原因被管理员杀死，或者遇上某种硬件故障，那么可能只有部分事务命令会被成功写入到磁盘中。
> 
> 如果 Redis 在重新启动时发现 AOF 文件出了这样的问题，那么它会退出，并汇报一个错误。
> 
> 使用 redis-check-aof 程序可以修复这一问题：它会移除 AOF 文件中不完整事务的信息，确保服务器可以顺利启动。

但是在另一篇文章写到 Redis 的事务不是原子性的，他强调的是 Redis 事务在执行失败的时候不会进行任何重试或回滚，因此不具备原子性。

使用事务可能会遇到以下两种错误。

  * 事务在执行 EXEC 之前，入队的命令可能会出错。比如说，命令可能会产生语法错误（参数数量错误，参数名错误，等等），或者其他更严重的错误，比如内存不足（如果服务器使用 maxmemory 设置了最大内存限制的话）。
  * 命令可能在 EXEC 调用之后失败。举个例子，事务中的命令可能处理了错误类型的键，比如将列表命令用在了字符串键上面，诸如此类。

示例：

```
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.

MULTI
+OK

SET a 3
abc

+QUEUED
LPOP a

+QUEUED
EXEC

*2
+OK
-ERR Operation against a key holding the wrong kind of value
```

对于 EXEC 执行之前的错误，Redis 会检查出来并返回错误自动放弃事务，但是对于在 EXEC 调用后执行失败的情况，该条语句会执行失败，但事务中的其他命令仍会执行。

因此严格来说，Redis 事务确实不具备原子性的特征。

### Redis 为什么不支持回滚

> 如果你有使用关系式数据库的经验， 那么 “Redis 在事务失败时不进行回滚，而是继续执行余下的命令”这种做法可能会让你觉得有点奇怪。
> 
> 以下是这种做法的优点：
> 
>   * Redis 命令只会因为错误的语法而失败（并且这些问题不能在入队时发现），或是命令用在了错误类型的键上面：这也就是说，从实用性的角度来说，失败的命令是由编程错误造成的，而这些错误应该在开发的过程中被发现，而不应该出现在生产环境中。
>   * 
>   * 因为不需要对回滚进行支持，所以 Redis 的内部可以保持简单且快速。
> 
> 有种观点认为 Redis 处理事务的做法会产生 bug ， 然而需要注意的是， 在通常情况下， 回滚并不能解决编程错误带来的问题。 举个例子， 如果你本来想通过 INCR 命令将键的值加上 1 ， 却不小心加上了 2 ， 又或者对错误类型的键执行了 INCR ， 回滚是没有办法处理这些情况的。
> 
> 鉴于没有任何机制能避免程序员自己造成的错误， 并且这类错误通常不会在生产环境中出现， 所以 Redis 选择了更简单、更快速的无回滚方式来处理事务。

**本文是对以下参考资料的整理。**

#### 参考资料

<a href="http://redisdoc.com/topic/transaction.html" target="_blank" rel="noopener nofollow">http://redisdoc.com/topic/transaction.html</a>

<a href="http://redisbook.readthedocs.io/en/latest/feature/transaction.html" target="_blank" rel="noopener nofollow">http://redisbook.readthedocs.io/en/latest/feature/transaction.html</a>

<a href="https://zh.wikipedia.org/wiki/ACID" target="_blank" rel="noopener nofollow">https://zh.wikipedia.org/wiki/ACID</a>