---
title: 常用的CentOS 7 yum源集合
author: 深 呼吸
type: post
date: 2016-03-31T09:40:55+00:00
url: /2016/03/centos-7-repo/
duoshuo_thread_id:
  - 6268149401944851202
categories:
  - 服务器

---
记录几个常用的CentOS 7下的yum源，包括PHP7，MariaDB，Redis，Nginx等，以及阿里云源，方便虚拟机或云主机上安装这些软件。

<!--more-->

### 1.PHP7 remi源

使用remi源：

<code class="lang:sh decode:true ">$ sudo rpm --import http://rpms.famillecollet.com/RPM-GPG-KEY-remi

$ sudo rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm</code>

### 2.MariaDB 10.1

Here is your custom MariaDB YUM repository entry for CentOS. Copy and paste it into a file under /etc/yum.repos.d/ (we suggest naming the file MariaDB.repo or something similar).

<code class="lang:sh decode:true "># MariaDB 10.1 CentOS repository list - created 2016-03-31 09:25 UTC
# http://mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.1/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1</code>

执行安装命令：

<code class="lang:sh decode:true">sudo yum install MariaDB-server MariaDB-client</code>

### 3.Redis

<code class="lang:sh decode:true ">wget http://download.redis.io/redis-stable.tar.gz
tar xvzf redis-stable.tar.gz
cd redis-stable
make</code>

设置service方式启动，参见：<a href="http://redis.io/topics/quickstart" target="_blank" rel="nofollow">quickstart</a>和<a href="https://blog.tanteng.me/2016/03/centos-redis-server/" target="_blank">Redis生成环境自动启动</a>

### 4.Nginx 1.8

To add NGINX yum repository, create a file named <code class="docutils literal">&lt;span class="pre">/etc/yum.repos.d/nginx.repo&lt;/span></code> and paste one of the configurations below:

CentOS:

<code class="lang:sh decode:true ">[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1</code>

### 5.阿里云

#### ① 备份

<code class="lang:sh decode:true ">mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup</code>

#### ② 下载新的CentOS-Base.repo 到/etc/yum.repos.d/

CentOS 7

<code class="lang:sh decode:true ">wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo</code>

#### ③ 之后运行yum makecache生成缓存

小谈博客整理，转载请注明出处和链接。本文链接：<a href="https://blog.tanteng.me/2016/03/centos-7-repo/" target="_blank">https://blog.tanteng.me/2016/03/centos-7-repo/</a>