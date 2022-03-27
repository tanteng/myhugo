---
title: Linux 中 /var/spool/postfix/maildrop 占用空间很大的原因
author: 小谈
type: post
date: 2017-01-14T05:52:21+00:00
url: /2017/01/linux-postfix-maildrop/
duoshuo_thread_id:
  - 6375334105399165698
categories:
  - 服务器

---
MySQL 报错 Exception Message:SQLSTATE\[08004\]\[1040\]Too many connections，经查这次错误是硬盘空间满了导致的，于是找一些可以删除的文件腾出一些空间。

<!--more-->

### 空间占用大的原因

发现 /var/spool/postfix/maildrop 这个目录占用了 6G 多的空间，网上找到一样的问题，原因是：

> 由于 Linux 在执行 cron 时，会将 cron 执行脚本中的 output 和 warning 信息，都会以邮件的形式发送 cron 所有者， 而由于客户环境中的 sendmail 和 postfix 没有正常运行，导致邮件发送不成功，全部小文件堆积在了 maildrop 目录下面，而且没有自动清理转换的机制，所以长达一年的时间，此目录已堆积了大量的文件。查看 man cron 的信息，可以知道会发送给 cron owner.

于是尝试删除这个目录下的内容，但是执行 rm -rf ./* 竟然提示参数列表过长，后来使用如下命令删除：

ls | xargs rm -f

通过管道的方式删除。

### 脚本重定向输出

所以注意在 crontab 脚本输出内容到日志，或者 /dev/null 2>&1，避免产生大量不必要的文件。

#### 几个 Linux 查找文件和空间的命令

find . -type f -size +1000000k 查找大文件和目录

du -s * | sort -nr | head 显示前十个占用空间最大的文件或目录

du -sh * 遍历目录大小

df -hl 系统各挂载硬盘空间大小