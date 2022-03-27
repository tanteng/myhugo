---
title: Mac 上如何打开 /usr/local 路径的文件夹
author: 深 呼吸
type: post
date: 2017-11-29T14:56:23+00:00
url: /2017/11/mac-open-user-local/
categories:
  - Develop
  - Go

---
在 Mac 的 Sublime 或者 Visual Studio Code 中选择打开文件夹，无论如何也无法选择 /usr/local 路径，在 Finder 中还可以使用“前往文件夹”输入路径，不过有解决办法，在终端使用命令行的方式启动软件打开指定文件夹，如：

比如打开 golang 标准库的 src 文件夹。

命令：

`open -a Visual\ Studio\ Code /usr/local/go/src/`

这样就成功运行 Visual Studio Code 软件并打开这个文件夹了。