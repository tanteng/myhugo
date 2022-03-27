---
title: 巧用vagrant建立映射目录解决环境兼容问题
author: 小谈
type: post
date: 2016-04-08T14:12:34+00:00
url: /2016/04/vagrantfile-synced-folder/
duoshuo_thread_id:
  - 6271188113628857090
categories:
  - Develop
  - PHP
  - 服务器

---
使用Vagrant装虚拟机的童鞋应该知道在Vagrantfile文件中可以定义本机和虚拟机目录映射，如：

`config.vm.synced_folder "../website", "/usr/share/nginx/html"`

表示把本机的当前文件目录下website目录映射到虚拟机的指定目录，这样在虚拟机中该目录的内容即是website目录下的，也就是实现了目录共享。

<!--more-->

本文不介绍如在vagrantfile中指定同步目录，而是使用目录映射的方式解决一个开发环境和本地环境兼容问题。

如在测试服务器上项目的代码：

<code>//added by maqg 20100317 请根据情况配置 vip_center 和 admin_vip路径
$system_folder_vip_prefix = "/data/vhosts/dynamic.xxx.com/";
$system_folder_admin_prefix = "/data/vhosts/admin.vip_center/";</code>

这是一个很老的项目，从代码注释看09年就开始搭建了，由于历史原因，许多地方很不规范。这里的路径都是写死的，许多地方都是这样，而且测试服务器文件组织很乱，有的用软链接链来链去，本地不可能按照服务器目录建一套一样的，那么用vagrantfile同步目录可以巧妙解决这个问题。

比如测试服务器上dynamic.xxx.com目录的文件，那么本地是在bbb目录中，这样可以在vagrantfile文件中建立一个映射目录，把dynamic.xxx.com目录映射为本机的bbb目录，这样在虚拟机中还是原来的路径，但是可以访问本机的bbb目录，不是可以不用修改代码路径解决问题吗？

同理，在其他地方也建立这样的映射目录，可以不修改代码，实现代码在不同环境下的兼容。

在虚拟机同样也可以使用软链接的方式，但如果使用的是vagrant装的虚拟机，用这种方式更好。