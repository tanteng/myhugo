---
title: MySQL数据库性能优化之缓存参数设置
author: 深 呼吸
type: post
date: 2016-03-28T04:02:15+00:00
url: /2016/03/innodb_buffer_pool_size/
duoshuo_thread_id:
  - 6266948869439881985
categories:
  - MySQL
  - 数据库
  - 服务器

---
网站运行在阿里云上，1G内存，PHP7+PHP-FPM+Nginx+MariaDB+Redis都安装在一台服务器上，而网站访问量一天也有500IP，不多，但也造成了一点压力，刚放上去几天数据库经常会挂掉，于是查阅数据库方面的性能优化，需要设置一些参数。

<!--more-->

无论是对于哪一种数据库来说，缓存技术都是提高数据库性能的关键技术，物理磁盘的访问速度永远都会与内存的访问速度永远都不是一个数量级的。通过缓存技术无论是在读还是写方面都可以大大提高数据库整体性能。

### MariaDB配置文件修改

配置文件路径：/etc/my.cnf.d/server.cnf，修改如下，添加innodb\_buffer\_pool\_size和max\_allowed_packet两个参数，暂时解决了数据库会挂掉的问题。

<code class="lang:php decode:true "># this is only for the mysqld standalone daemon
[mysqld]
log_error=/var/log/mariadb_error.log
innodb_buffer_pool_size=164M
max_allowed_packet = 10M</code>

### innodb\_buffer\_pool_size参数

Innodb存储引擎的缓存机制和MyISAM的最大区别就在于Innodb不仅仅缓存索引，同时还会缓存实际的数据。所以，完全相同的数据库，使用Innodb存储引擎可以使用更多的内存来缓存数据库相关的信息，当然前提是要有足够的物理内存。这对于在现在这个内存价格不断降低的时代，无疑是个很吸引人的特性。

innodb\_buffer\_pool\_size参数用来设置Innodb最主要的 Buffer(Innodb\_Buffer_Pool)的大小，也就是缓存用户表及索引数据的最主要缓存空间，对Innodb整体性能影响也最大。无论是 MySQL 官方手册还是网络上很多人所分享的Innodb优化建议，如果是专用服务器，可以设置为整个系统物理内存的50%～80%之间。

### max\_allowed\_packet参数

如果一条sql语句很长，或者包含BLOB数据，会造成内存溢出。在以上配置文件中添加或者修改以下变量：

max\_allowed\_packet = **10M** (也可以设置自己需要的大小)

max\_allowed\_packet 参数的作用是，用来控制其通信缓冲区的最大长度。

### 其他的一些配置参数

#### **innodb\_buffer\_pool_size**

如果用Innodb，那么这是一个重要变量。相对于MyISAM来说，Innodb对于buffer size更敏感。MySIAM可能对于大数据量使用默认的key\_buffer\_size也还好，但Innodb在大数据量时用默认值就感觉在爬了。 Innodb的缓冲池会缓存数据和索引，所以不需要给系统的缓存留空间，如果只用Innodb，可以把这个值设为内存的70%-80%。和 key_buffer相同，如果数据量比较小也不怎么增加，那么不要把这个值设太高也可以提高内存的使用率。

#### **innodb\_additional\_pool_size**

这个的效果不是很明显，至少是当操作系统能合理分配内存时。但你可能仍需要设成20M或更多一点以看Innodb会分配多少内存做其他用途。

#### **innodb\_log\_file_size**

对于写很多尤其是大数据量时非常重要。要注意，大的文件提供更高的性能，但数据库恢复时会用更多的时间。我一般用64M-512M，具体取决于服务器的空间。

#### **innodb\_log\_buffer_size**

默认值对于多数中等写操作和事务短的运用都是可以的。如 果经常做更新或者使用了很多blob数据，应该增大这个值。但太大了也是浪费内存，因为1秒钟总会 flush（这个词的中文怎么说呢？）一次，所以不需要设到超过1秒的需求。8M-16M一般应该够了。小的运用可以设更小一点。

#### **innodb\_flush\_log\_at\_trx_commit （这个很管用）**

抱怨Innodb比MyISAM慢 100倍？那么你大概是忘了调整这个值。默认值1的意思是每一次事务提交或事务外的指令都需要把日志写入（flush）硬盘，这是很费时的。特别是使用电 池供电缓存（Battery backed up cache）时。设成2对于很多运用，特别是从MyISAM表转过来的是可以的，它的意思是不写入硬盘而是写入系统缓存。日志仍然会每秒flush到硬 盘，所以你一般不会丢失超过1-2秒的更新。设成0会更快一点，但安全方面比较差，即使MySQL挂了也可能会丢失事务的数据。而值2只会在整个操作系统挂了时才可能丢数据。

#### 参考链接：

  * <a href="http://blog.csdn.net/yang1982_0907/article/details/20123055" target="_blank" rel="nofollow">MySQL的InnoDB缓存相关优化</a>
  * <a href="http://www.educity.cn/wenda/400332.html" target="_blank" rel="nofollow">MySQL配置项</a>
  * <a href="http://database.51cto.com/art/201503/470310.htm" target="_blank" rel="nofollow">MariaDB配置文件详解</a>