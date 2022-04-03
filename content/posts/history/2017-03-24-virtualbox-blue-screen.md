---
title: VirtualBox启动蓝屏问题
author: 深 呼吸
type: post
date: 2017-03-24T11:18:09+00:00
url: /2017/03/virtualbox-blue-screen/
categories:
  - 服务器

---
用 vagrant + VirtualBox 虚拟机，最近几次没有关闭虚拟机重启电脑，导致虚拟机无法启动，每次到 Booting VM 这个步骤就会蓝屏，重新安装 vagrant，VirtualBox 软件都无法正常运行。

后来在网上找到一个可行的方法，在“启用或关闭Windows功能”中把 Hyper-V 特性关闭并重启电脑，再重新安装虚拟机可以正常启动了。

不明白具体的原因，但暂时通过这个方法解决问题。设备：Win 10.